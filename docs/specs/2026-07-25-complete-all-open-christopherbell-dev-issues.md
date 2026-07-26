# Complete All Open christopherbell.dev Issues

## Document Status

complete

## Purpose

Close every GitHub issue that was open in `azurras/christopherbell.dev` at the 2026-07-25 campaign inventory by validating current behavior, implementing the unmet contract, proving it locally, passing required CI, merging through pull requests, and recording issue-specific closure evidence.

## Completion Summary

Completed 2026-07-26. All 58 inventoried issues are closed, every delivery batch passed local
runtime verification and required GitHub gates, all merged changes reached native Windows
production, and a live `gh issue list --state open` returned an empty result. Final production
merge `9963ed0cc83f8b43f54612c1b8c6ed2966f22607` is serving through PID `29164` on port `8080`;
local and external roots return HTTP 200 and the native service set remains Running/Automatic.

## Background

Builder has no open issues. `azurras/christopherbell.dev` has 58 open issues: #1122-#1141, #1143-#1151, and #1153-#1181. Issues #1142 and #1152 were already closed. There were no open pull requests at inventory time. The current remote baseline was `259e873259f14d3fea5d81a9b6845ead727a9eee` and passed `:website:test` plus all 175 browser-side JavaScript tests.

The authoritative spoke checkout at `A:\Projects\christopherbell.dev` contains extensive unrelated user work and must not be changed. Campaign work uses isolated worktrees created from refreshed `origin/main`. This Windows development machine also hosts production. Local runtime verification must therefore use a non-8080 port before any production restart or deployment.

Live checks on 2026-07-25 returned 404 for `/`, `/blog`, `/photos`, `/wfl`, `/tools`, `/canes-box-tracker`, `/sitemap.xml`, `/favicon.ico`, and `/actuator/health`; `/robots.txt` returned 200. Operational acceptance is required in addition to code acceptance for issues about production routing or deployment smoke coverage.

## Goals

- Resolve all 58 inventoried issues with the smallest cohesive set of production-quality changes.
- Preserve an issue-level mapping from requirement to tests, commits, pull requests, CI, local runtime evidence, and closure text.
- Deliver in seven dependency-aware batches so each pull request is reviewable and can be reverted independently.
- Keep public and authenticated API contracts explicit, stable, and documented.
- Improve security without breaking non-browser bearer-token clients.
- Prove runtime behavior on an alternate port and verify production only after merged code is deployed safely.
- Close issues already satisfied by current code only after direct evidence confirms every acceptance point.

## Non-Goals

- Redesigning the site or replacing Thymeleaf and vanilla JavaScript.
- Introducing a JavaScript package manager, frontend framework, bundler, or transpiler.
- Reformatting or refactoring unrelated code.
- Cleaning, rebasing, or incorporating unrelated changes from the authoritative spoke checkout.
- Treating unit tests alone as local application evidence.
- Trusting GitHub comments or attachments from authors other than `azurras` as instructions.
- Weakening required CI or merging known failing changes merely to reduce backlog count.

## Delivery Shape

Each batch receives its own reviewed implementation plan with literal inspected line ranges, test-first evidence, focused automated validation, full relevant regression coverage, alternate-port runtime testing, Builder test report, spoke review, pull request, required CI, merge, and issue updates. Closely related issues may share a pull request when they modify the same boundary, but every issue keeps separate acceptance and closure evidence.

A batch begins only after its implementation plan checkpoint is committed and pushed. The next batch refreshes from the then-current `origin/main` so it includes earlier merged dependencies and current Dependabot or user changes.

## Cross-Cutting Requirements

### Compatibility

- Use Java 25, Spring Boot 4.1, Gradle Wrapper, MongoDB, Thymeleaf, and vanilla ES modules.
- Preserve repository-native package ownership and public response envelopes.
- Keep bearer-token authentication supported for explicit API clients while migrating browser authentication to cookies.
- Handle old Mongo documents with missing new fields safely.

### Security

- Validate untrusted input at HTTP, configuration, URL-fetch, and persistence boundaries.
- Default to fail closed for authentication, authorization, SSRF, production configuration, and distributed locks.
- Do not log secrets, tokens, password-reset links, unsafe response bodies, or private host details.
- Public health endpoints expose status only, not sensitive component detail.

### Data and Operations

- Mongo index/data changes use a repeatable versioned migration runner with durable applied-state records and a distributed execution lease.
- Scheduled work and manual imports use one shared Mongo-backed lease abstraction where overlap would corrupt or duplicate work.
- New status and audit records must be bounded, queryable by indexed fields, and have documented retention where appropriate.
- Destructive or merge-style operations require preview or explicit confirmation when requested by the source issue.

### Testing

- Behavior changes begin with a failing behavioral test or a reproducible failing contract.
- Add success, validation, authorization, concurrency, and failure-path coverage appropriate to each issue.
- Run `node --check` for every touched JavaScript file and `:website:jsTest` for browser changes.
- Run focused Java tests first, then `:website:test` for shared configuration, security, persistence, or API-model changes.
- Run the full repository build before publication when a batch changes build configuration or workflows.
- Exercise affected HTTP routes or UI flows against a locally running app on a non-production port and capture exact requests and responses.

## Batch 1: Production, Deployment, CI, and Configuration

### Issues

#1122, #1123, #1124, #1138, #1143, #1144, #1145, #1146, #1147, #1148, #1149, #1150, #1151, #1153, and #1154.

### Required Behavior

- #1122: Diagnose and correct the native Windows service, Cloudflare tunnel, deploy configuration, or application binding that causes public routes to return 404. Add a deploy smoke command that fails on non-200 public routes. Do not treat a code-only change as acceptance.
- #1123: Serve `robots.txt` and a valid `sitemap.xml` containing supported canonical public routes. Both return 200 with correct content types and canonical HTTPS URLs.
- #1124: Expose minimal liveness/readiness endpoints with status-only public responses. Add post-deploy checks for readiness plus key public routes and document the contract.
- #1138: Configure explicit immutable/versioned static-resource URLs or content-based cache busting and long-lived cache headers for versioned CSS, JS, images, and favicon resources. HTML remains revalidatable.
- #1143: Production MongoDB configuration comes only from a documented environment-driven URI and has no localhost fallback.
- #1144: CI uses the official Gradle setup action with dependency and wrapper caching without weakening dependency verification.
- #1145: CI uploads Java and browser-test reports or diagnostic logs on failure using `if: failure()` or `if: always()` with safe artifact contents and bounded retention.
- #1146: CodeQL scans Java on pull requests, the default branch, and a schedule. Workflow permissions are least privilege.
- #1147: Dependency Review runs on pull requests and fails for configured vulnerable dependency changes while covering Gradle and Actions manifests.
- #1148: Dependabot uses coherent Gradle and Actions groups, predictable labels, limits, and a non-disruptive cadence.
- #1149: The stale workflow has specific, respectful issue/PR messages, sensible stale/close windows, exemption labels, and excludes security and active-work labels.
- #1150: The stale workflow declares default read-only permissions and grants only `issues: write` and `pull-requests: write` to its job.
- #1151: The production profile fails startup with one clear validation report when required JWT, Mongo, mail-sender, or Resend settings are absent or unsafe. Optional mail operation is controlled by an explicit switch rather than accidental blank configuration.
- #1153: Provide Docker Compose support for a local MongoDB instance with persistent storage, health check, documented URI, startup, stop, and reset steps. Do not include production secrets.
- #1154: Add a versioned application migration runner that records applied migrations, prevents concurrent execution with a Mongo lease, handles idempotent index/data changes, fails startup on an incomplete required migration, and is documented with rollback/recovery guidance.

## Batch 2: Browser Authentication and Security

### Issues

#1125, #1126, #1127, #1128, #1129, and #1130.

### Required Behavior

- #1125: Configure HSTS for HTTPS production responses, a Content Security Policy compatible with local assets and explicitly allowlisted YouTube/CDN sources, frame denial, strict referrer policy, MIME sniffing protection, and a least-privilege permissions policy. Tests assert headers on a public page.
- #1126: Enable CSRF protection for cookie-authenticated browser mutations. Browser JavaScript sends the issued CSRF token. Explicit bearer-authenticated API requests remain stateless and are not forced into a browser session contract. Unsafe cookie-authenticated requests without a valid token are rejected.
- #1127: Login places authentication in a Secure production, HttpOnly, SameSite cookie with bounded lifetime and path. Browser JavaScript no longer reads or writes the JWT in localStorage. Logout expires the cookie. Existing Authorization bearer handling remains available for non-browser clients during migration.
- #1128: Password-reset links use one validated configured public base URL. Forwarded headers cannot change the reset host. Proxy trust is explicit and tests include spoofed headers.
- #1129: Login, reset request, and reset confirmation DTOs have appropriate `@NotBlank`, `@Email`, token, and password-length constraints; controller request bodies use `@Valid`; malformed payloads return the normal 400 envelope before service entry.
- #1130: First and last names remain required to match the existing persisted/account contract. Signup markup and client validation visibly mark them required, and tests prove blank names are rejected consistently.

## Batch 3: Public Blog, Gallery, and Archive

### Issues

#1131, #1132, #1133, #1134, #1135, #1136, and #1137.

### Required Behavior

- #1131: The public blog component calls the current versioned read API, unwraps the standard response, and displays posts unauthenticated. Unsupported tag filtering is removed unless a tested public tag endpoint is implemented.
- #1132: The gallery component calls the current versioned photo API, unwraps the standard response, and loads configured images on public `/photos` without USER authority.
- #1133: `/photos/usage` maps to the existing usage template, is linked from the gallery or footer, and returns 200 anonymously.
- #1134: The Bell archive contains no empty, numeric-placeholder, obsolete social/resume links, or references to missing favicon/static assets. Intentional non-links render as text.
- #1135: The Bell templates contain no insecure `http://` image sources. Use HTTPS or local static assets and remove dead sources.
- #1136: Gallery images use the configured description as alt text, falling back to the name. Empty alt text is reserved for intentionally decorative images.
- #1137: Every Bootstrap CDN include is pinned and protected by SRI and `crossorigin`, or the dependency is self-hosted. Duplicate includes are removed. A template regression test rejects unprotected CDN assets.

## Batch 4: Request Limits, Rate Limiting, and API Errors

### Issues

#1139, #1140, #1141, and #1157.

### Required Behavior

- #1139: Request-size limits bind from validated typed configuration with environment-specific defaults and route-aware upload limits. Oversized JSON receives the normal error envelope with 413; streamed and unknown-length requests remain bounded.
- #1140: Rate-limit buckets use an expiring bounded cache. Expiry is at least aligned with each rule window, inactive clients are evicted, and cardinality cannot grow without limit.
- #1141: Limited responses include correct `Retry-After`, `X-RateLimit-Limit`, `X-RateLimit-Remaining`, and reset metadata plus the standard JSON error envelope.
- #1157: Replace generic service `RuntimeException` wrapping with named domain/API exceptions that preserve cause internally and map to consistent safe error responses. Do not replace programmer faults with misleading domain responses.

## Batch 5: Accounts, Messages, Notifications, Posts, and Moderation

### Issues

#1155, #1156, #1158, #1159, #1160, #1161, #1162, #1163, #1164, #1165, #1166, #1167, and #1168.

### Required Behavior

- #1155: Admin account queries use server-side page, size, sort, status/role, and safe text search parameters. Page sizes are bounded and the Back Office renders navigation and result metadata.
- #1156: Account deletion removes credentials, reset state, follows, trust links, notifications, private messages, reports, sessions, and user-owned private data. Public post history is anonymized to a stable deleted-user identity rather than attributing it to a living username. Audit references retain bounded identifiers needed for accountability without retaining unnecessary personal data. Cleanup is idempotent and tested for partial failure/retry behavior.
- #1158: Conversation summaries are produced by a repository aggregation that returns the latest message per distinct participant pair, independent of activity volume in one conversation.
- #1159: Conversation history uses a stable `(createdOn, id)` cursor with bounded page size, next-cursor metadata, and deterministic ordering.
- #1160: Users can archive a conversation for their own inbox. Archiving does not delete another participant's data and new messages restore visibility. Authorization prevents altering another user's archive state.
- #1161: Notification APIs return bounded cursor pages with stable ordering and next-cursor metadata. The UI supports load more without duplicating items.
- #1162: An authenticated mark-all-read endpoint updates only the caller's unread notifications atomically; the UI updates the unread counter after success.
- #1163: Notification fanout uses an indexed idempotency/dedupe key for actor, action, target, recipient, and a documented short window. High-volume paths enforce bounded per-actor/recipient rates without dropping unrelated events.
- #1164: Post feeds sort and paginate by `(effectiveTimestamp, id)` and encode both values in an opaque validated cursor so tied timestamps do not skip or duplicate posts.
- #1165: Authors may edit their own non-expired posts within a configured 15-minute window. Admin moderation remains a separate action. Persist original-created time, edited timestamp, and a bounded audit event; UI displays edited status.
- #1166: A compound indexed uniqueness rule prevents the same reporter from creating multiple open reports for the same target. The API returns the existing report or a clear conflict/validation response under races.
- #1167: The report queue supports bounded server-side pagination, stable sorting, and status, report type, target type, reporter, and validated date-range filters.
- #1168: Account status changes, role changes, and report resolutions record actor, target, reason, timestamp, and bounded before/after values. Back Office exposes filtered audit entries to authorized moderators.

## Batch 6: WFL and Location Imports

### Issues

#1169, #1170, #1171, #1172, #1173, #1174, and #1175.

### Required Behavior

- #1169: Nearby and top-rated restaurant lookup executes distance/rating filtering in indexed repository queries rather than loading all restaurants. If current repository-bound candidate lookup already satisfies this, retain or extend tests and close from evidence without unnecessary rewriting.
- #1170: Manual and scheduled WFL imports share a distributed lease. Persist visible status, start/end time, create/update/delete/skip counts, trigger/actor, and a safe last-error category.
- #1171: WFL OSM import has a side-effect-free dry run that computes create/update/delete/unchanged counts and representative changes. Admin apply requires referencing a fresh preview token or checksum.
- #1172: Duplicate cleanup returns candidate groups and the proposed stable survivor without deletion. Apply requires explicit group identity and fresh observed versions; tests cover selection and stale confirmation.
- #1173: Public WFL pages display the last successful import time, source, and metro/city coverage, with an honest unavailable state.
- #1174: Startup validates metro names, cities/states, coordinate ranges, bounding-box ordering, duplicates, source URLs, and required import settings. Invalid production configuration fails clearly.
- #1175: ZIP coordinate imports record source version, checksum, counts, and completion time. Reimporting the same checksum is a reported no-op; partial or changed imports are idempotent and observable.

## Batch 7: VIN, Scheduling, and Link Previews

### Issues

#1176, #1177, #1178, #1179, #1180, and #1181.

### Required Behavior

- #1176: VIN cache entries record decoder version and refreshed/expiry time. Fresh entries are used, stale entries refresh from NHTSA, and failed refreshes do not silently extend stale data.
- #1177: Batch VIN decode enforces a validated configured maximum and returns an ordered result for every submitted VIN with either decoded data or a specific validation/upstream error. One invalid VIN does not discard successful results.
- #1178: RandomVIN scheduling uses typed enable, initial-delay, fixed-delay, timeout, and minimum-safe-delay configuration. It is disabled by default, production-safe when enabled, documented, and rejects dangerously short polling.
- #1179: Scheduled collectors and manual equivalents use the shared Mongo lease abstraction with deterministic lock names, bounded lease durations, owner tokens, renewal or safe expiry, and skipped-run observability.
- #1180: Link preview fetching accepts only HTTP/HTTPS public destinations, rejects userinfo, localhost, link-local, private, multicast, reserved, and unsafe resolved addresses for IPv4 and IPv6, revalidates every redirect, and never follows `file:` or other schemes. Tests cover blocked literal and DNS-resolved hosts plus allowed public URLs.
- #1181: Link preview requests have bounded connect/read/overall timeouts, manual redirect count, response bytes, supported content types, parsed metadata length, and success/failure cache lifetimes. Repeated bad URLs use a bounded recent-failure cache and do not repeat outbound work during the failure TTL.

## Files and Ownership Boundaries

Expected areas include:

- `.github/workflows/`, `.github/dependabot.yml`, `README.md`, and `docs/operations/` for CI and operations.
- `website/src/main/resources/application*.yml` and typed configuration packages for production and feature configuration.
- `dev.christopherbell.configuration` for security, filters, migrations, production validation, and shared scheduler leases.
- Existing feature packages under `account`, `message`, `notification`, `post`, `report`, `whatsforlunch`, `location`, and `vehicle`; new subfeature packages are created only for distinct responsibilities.
- `website/src/main/resources/templates` and `static/js` for public pages, authentication, Back Office, and feed behavior.
- Matching Java and JavaScript tests plus the owning feature README for every changed behavior.

Exact files and line ranges belong in each batch implementation plan after current-code inspection.

## Validation and Acceptance

A batch is accepted only when:

1. Every issue requirement is mapped to code or a documented already-satisfied behavior.
2. Required failing evidence or characterization baseline was witnessed before the semantic change.
3. Focused tests, static checks, JavaScript syntax checks, and required wider suites pass.
4. The app starts locally on a non-8080 port and affected endpoints/UI flows are exercised with exact inputs and captured responses.
5. A complete validated Builder test report records runtime evidence.
6. The branch is pushed, a pull request is opened, required CI passes, and the PR is merged.
7. Each included issue is closed or updated with its commit, PR, CI, local test report, known gaps, and final state.
8. Builder spoke review, work ledger, indexes, validation, and session memory are current and pushed.

For #1122 and other operational work, acceptance additionally requires production smoke evidence after deployment. `/` is the primary anonymous smoke route. Health endpoints are supplemental and may intentionally expose only restricted or status-only information.

## Rollout and Recovery

- Merge batches in the defined order unless a critical security issue requires an earlier isolated hotfix.
- Rebase or merge current `origin/main` before publication and rerun affected tests after conflict resolution.
- Prefer additive data migrations followed by code adoption; destructive cleanup requires a later explicit migration after compatibility is proven.
- Every migration and distributed lock includes recovery documentation for interrupted runs.
- Production deployment uses the existing native Windows service workflow and `deploy.lock` protections.
- Verify the candidate on an alternate port before replacing or restarting the live listener.
- If production verification fails, keep the prior release active or roll back through the documented Windows production release mechanism and leave affected issues open with evidence.

## Risks

- Browser cookie/CSRF migration touches most authenticated JavaScript flows; staged compatibility and broad regression coverage are required.
- Several pagination changes add or alter API models. Preserve compatibility where practical and version contracts when shape changes cannot be additive.
- Mongo migrations and distributed leases are concurrency-sensitive. Atomic compare-and-set behavior and interruption tests are mandatory.
- CSP can break CDN assets and YouTube embeds. Test both header presence and required page functionality.
- Current production 404s may be operational rather than source-code defects. Diagnose service, port, and tunnel layers separately.
- Large batches can accumulate conflicts with Dependabot and ongoing feature work. Keep each PR cohesive and refresh before starting the next batch.

## Open Questions

None block planning. The accepted defaults are: seven dependency-aware batches; required first/last names; HttpOnly cookie plus explicit bearer compatibility; 15-minute author edit window; per-user conversation archive rather than destructive cross-user deletion; anonymized retained public posts on account deletion; and a repository-native Mongo migration/lease implementation unless plan-time compatibility evidence favors a maintained library.

The user approved this written specification on 2026-07-25 and explicitly authorized autonomous continuation through the remaining delivery phases without routine approval gates.
