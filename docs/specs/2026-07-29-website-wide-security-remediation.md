# Website-Wide Security Remediation

## Document Status

complete

## Purpose

Remediate every attacker-reachable finding from the complete security review of `azurras/christopherbell.dev` at immutable revision `f77c5f5bb644cc75cf98b27e722efdc00cd036f1` plus the complete follow-up security review of the ActivityPub delta through `6c1501070ff518bc040583c4576c2df201dcd3ed`, prove each security boundary with regression evidence, and deliver the result through PR, CI, merge, production-safe verification, and Builder closeout.

## Background

The repository-wide scan accounted for all 1,105 tracked files and produced 18 candidates. Validation and attack-path analysis retained 15 findings: five high, seven medium, and three low. Three operator/admin-only correctness candidates were suppressed because no untrusted principal could reach them. Baseline `:website:check`, GitHub CI, and CodeQL passed at the scanned revision.

Before implementation, refreshed `origin/main` added a 93-file ActivityPub federation delta. Every changed file received a full-file review receipt. Focused validation found three candidate gaps: the public outbox omitted the creation-time federation-eligibility flag, signup preselected federation consent instead of requiring affirmative opt-in, and federation predicates appeared to omit the independently mutable account approval state. Attack-path policy initially retained the outbox and approval candidates as two reportable low-severity findings and rejected the signup default from the security count because the clearly disclosed choice affects only the registering user. Before remediation began, current main intentionally removed the dormant approval field and made `AccountStatus.ACTIVE` authoritative, so the approval candidate became not applicable rather than a fix target. The outbox gap remains reportable, and the user-approved remediation still includes changing signup to affirmative opt-in as privacy hardening. The outbound transport, DNS pinning, request signing, encrypted-key handling, retry bounds, and configuration gates survived review.

The dirty authoritative checkout at `A:\Projects\christopherbell.dev` must remain untouched. Work executes in `A:\Projects\christopherbell.dev-worktrees\security-audit-20260728` on branch `codex/security-audit-20260728`, based on refreshed `origin/main`.

## Goals

- Make bearer authorization reflect current account validity, role, password, and permission state immediately.
- Remove account-state enumeration while preserving useful bounded internal diagnostics.
- Store password verifiers in an explicit upgradeable format and migrate legacy verifiers after successful authentication.
- Prevent link-preview DNS rebinding and active non-HTTP(S) preview links.
- Enforce creator authority and bounded membership for shared WFL sessions.
- Prevent unsafe restaurant website schemes at ingestion and rendering boundaries.
- Prevent upload-resume metadata from crossing account boundaries in a shared browser.
- Return stable public error contracts without stack-trace amplification for routine client failures.
- Make the production trusted-proxy chain explicit and fail startup for invalid CIDRs.
- Require affirmative federation enrollment, enforce current active status throughout federation, and preserve the creation-time per-post publication boundary in the public outbox.
- Pin executable build inputs through immutable GitHub Action SHAs, a Gradle distribution checksum, and strict Gradle dependency verification.
- Preserve current public behavior except where it is unsafe, add regression tests first, and complete alternate-port and production verification.

## Non-Goals

- Do not change command-center property validation or asynchronous power-action result semantics; those candidates are privileged operational correctness concerns rather than attacker-reachable security flaws.
- Do not change administrative WFL account-deletion cascade behavior in this security remediation; self-service deletion does not exist.
- Do not weaken production filesystem ACLs, disable CSRF, broadly trust forwarding headers, or touch unrelated dirty checkout state.
- Do not add unrelated feature work or broad refactors.

## Requirements

### Authentication and password security

- Bearer authentication must reload the current account before granting authority.
- Tokens must carry an account security version; missing or stale versions must fail closed.
- Password resets and changes to role, active status, or resource permissions must advance the security version.
- Public login failures for unknown account, wrong password, and inactive account must have the same status and response shape.
- Unknown-account login must perform comparable password-verification work to reduce timing distinction.
- New password verifiers must use the current versioned PBKDF2-HMAC-SHA256 format with 210,000 iterations, a per-password random salt, and constant-time byte comparison.
- Existing 65,536-iteration salt/hash records must remain verifiable and be upgraded after a successful login.

### HTTP boundary and client identity

- Framework parsing, binding, and response-status failures must expose stable safe messages rather than raw framework exception text.
- Routine client failures must not create ERROR stack traces; unexpected server failures must retain causal logging and a stable public response.
- Trusted proxy entries must be syntactically validated IPv4 or IPv6 CIDRs during startup.
- Production must trust only the explicitly configured local proxy hops, with a safe loopback default suitable for the local tunnel topology.
- Direct requests and requests from untrusted peers must ignore forwarding headers.

### WFL authorization, capacity, and URLs

- Only `createdByAccountId` may replace a session's restaurant choices or clear votes.
- The detail response must expose a creator capability used by the UI to hide creator-only controls for participants.
- The server rule remains authoritative if a caller bypasses the UI.
- A session must have no more than 21 account participants: one creator plus at most 20 joined/invited participants.
- Concurrent joins must not exceed the cap or create duplicate membership.
- Restaurant website URLs must be absolute HTTP or HTTPS URLs at administrative create/update and import boundaries.
- Public renderers must defensively suppress unsafe or legacy stored values.

### Link previews and browser state

- Link-preview destination approval and network connection must use the same approved IP address on every request and redirect.
- The original hostname must remain the HTTP Host and TLS hostname-verification identity.
- Loopback, private, link-local, multicast, unspecified, and otherwise non-public addresses must remain blocked for literals and DNS answers.
- Preview image metadata must be limited to absolute HTTP or HTTPS URLs before persistence/response.
- Renderer fallbacks must also suppress non-HTTP(S) values.
- Upload resume records must be keyed by a non-secret current-account identifier, must not read the legacy global key, and must clear on terminal completion or explicit discard.

### Build supply chain

- Every external GitHub Action reference must use the full reviewed commit SHA and retain the intended release tag in a comment.
- `gradle-wrapper.properties` must contain the authoritative checksum for its exact Gradle distribution.
- `gradle/verification-metadata.xml` must contain reviewed SHA-256 checksums for all resolved build, plugin, test, packaging, and sensor-runtime artifacts.
- CI must execute dependency verification in strict mode, and the repository must document the intentional metadata-update workflow.

### Federation privacy and moderation

- Signup federation enrollment must default to unchecked and must become enabled only after an explicit affirmative choice.
- Consent copy must accurately describe public identity, posts, and local relationship collections without implying that enrollment is automatic.
- Federation enrollment, discovery, creation-time publication eligibility, queued delivery, and live delivery must require `AccountStatus.ACTIVE`; no retired approval field may be reintroduced.
- Changing an account away from `ACTIVE` must make public federation discovery fail closed and prevent subsequent delivery without relying on a separate consent mutation.
- Public ActivityPub outbox page and count queries must include only posts whose persisted `federationOutboundEligible` value is explicitly true.
- Historical null or false eligibility values must remain excluded even if the account enables federation later.

## Proposed Approach

Deliver one integrated PR with cohesive commits grouped by boundary:

1. Introduce account security-version state, current-account bearer lookup, uniform login failure, and versioned password verification/migration.
2. Harden the public error contract and trusted-proxy configuration.
3. Enforce WFL creator authority and an atomic/optimistic bounded membership transition, then centralize safe restaurant website parsing for ingestion and rendering.
4. Introduce an outbound preview transport seam that resolves once and connects only to the approved address while retaining the original origin identity; centralize safe preview image URL parsing.
5. Namespace upload resume state by current account.
6. Make federation enrollment affirmative, preserve authoritative active-account checks, and enforce creation-time post eligibility for public outboxes.
7. Pin workflows and Gradle inputs, then run focused, full, packaged-runtime, alternate-port, and production checks.

Each behavioral change begins with a failing public-boundary test. Security checks live inside service, authentication, parsing, or transport boundaries rather than only in controllers or UI code. Compatibility adapters are limited to legacy password verification and defensive rendering of existing URL data.

## Files and Modules Involved

- `website/src/main/java/dev/christopherbell/account/**`
- `website/src/main/java/dev/christopherbell/auth/**`
- `website/src/main/java/dev/christopherbell/config/**`
- `website/src/main/java/dev/christopherbell/error/**`
- `website/src/main/java/dev/christopherbell/restaurant/**`
- `website/src/main/java/dev/christopherbell/wfl/**`
- `website/src/main/java/dev/christopherbell/post/preview/**`
- `website/src/main/resources/static/js/**`
- `website/src/main/resources/application-prod.yml`
- corresponding Java and JavaScript tests
- `website/src/main/java/dev/christopherbell/federation/**`
- `website/src/main/resources/templates/signup.html`
- `.github/workflows/**`
- `gradle/wrapper/gradle-wrapper.properties`
- `gradle/verification-metadata.xml`
- build/dependency documentation

## Validation Plan

- Capture RED and GREEN evidence for each finding at the narrowest public boundary.
- Run focused Java and JavaScript tests after each remediation group.
- Run `node --check` for changed JavaScript.
- Run strict Gradle dependency verification and the full `:website:check` suite from an isolated `GRADLE_USER_HOME`.
- Build and run the packaged production-profile JAR on a non-8080 port.
- Exercise anonymous, authentication, authorization-denial, unsafe-URL, client-IP, and link-preview boundaries without touching the live listener.
- Review the final diff against the scan findings and code-writing rubric.
- Open a PR, wait for CI, Dependency Review, and CodeQL, merge only when green, and verify production through listener, service, and endpoint evidence without weakening ACLs.

## Acceptance Criteria

- All 16 surviving reportable findings have a corresponding merged fix and regression evidence; the retired approval-state candidate is documented as not applicable, and the approved self-only federation signup hardening has its own regression evidence without being presented as a reportable vulnerability.
- No suppressed candidate is presented as remediated security work.
- Focused tests and full `:website:check` pass with zero failures.
- Strict dependency verification and the Gradle wrapper checksum succeed from a clean isolated Gradle home.
- Alternate-port runtime checks pass before any production listener change.
- PR checks pass, the PR is merged, relevant trusted-owner GitHub issues are closed or updated with evidence, and production acceptance succeeds.
- Builder work record, test report, spoke review, closure, session memory, indexes, and validation are complete and pushed.

## Risks and Mitigations

- Stronger password hashing can increase login CPU cost; keep the 210,000-iteration verifier below one second on the production host and retain login rate limiting.
- Per-request account lookup adds bearer latency; use a single indexed account-ID lookup and preserve fail-closed behavior.
- DNS pinning can break HTTPS if origin identity is replaced by the IP; preserve the original hostname for Host/SNI/hostname verification and test redirects.
- Concurrent WFL joins can exceed a naive read-then-save cap; use an atomic conditional update or optimistic version retry and concurrency evidence.
- Dependency metadata can omit rarely resolved configurations; generate it using the full verification task set and rerun from a clean Gradle home.
- A production restart on the development host can affect live traffic; validate the exact artifact on an alternate port first and use existing guarded deployment automation.

## Open Questions

None. Implementation-level discoveries that invalidate an assumption require updating this specification and plan before proceeding.
