# Complete christopherbell.dev Issues 1258-1307

## Document Status

ready-for-execution

## Purpose

Close every GitHub issue currently open in `azurras/christopherbell.dev` by validating its evidence against refreshed mainline, implementing the unmet contract, proving behavior locally, passing required CI, merging reviewable pull requests, deploying safely on the native Windows production host, and recording issue-specific closure evidence.

## Background

The prior 58-issue campaign is complete. A subsequent evidence-backed audit created exactly 50 issues, #1258-#1307, and GitHub still reports all 50 open on 2026-07-29. Every issue was authored by `azurras`, has a non-empty body, and has zero comments. The only open pull request is unrelated Dependabot PR #1310.

The authoritative checkout at `A:\Projects\christopherbell.dev` is ahead 3, behind 90, and contains extensive unrelated user work. It must remain untouched. Delivery uses `A:\Projects\christopherbell.dev-worktrees\all-open-issues-20260729`, created from refreshed `origin/main` commit `8405cd77d0f1743fe33d70cc80b47e37048090a0`.

The development machine is also the native Windows production host. Runtime validation must use a non-8080 port before any live listener change. `/` is the reliable anonymous smoke endpoint; `/actuator/health` may correctly return 403.

## Goals

- Resolve all 50 inventoried issues with production-quality changes and no unrelated refactoring.
- Preserve issue-level traceability to tests, runtime evidence, commits, pull requests, CI, merges, deployment checks, and closure text.
- Deliver seven cohesive, independently reviewable batches in dependency order.
- Use bounded, concurrency-safe data models and migrations for account, feed, WFL, shared-folder, and command-center state.
- Improve public metadata, accessibility, build reproducibility, CI reliability, and software-supply-chain integrity.
- Close already-satisfied issues only after direct evidence proves every acceptance statement.

## Non-Goals

- Redesigning the product or replacing Spring Boot, Thymeleaf, MongoDB, Gradle, or vanilla JavaScript.
- Cleaning, rebasing, staging, or incorporating the authoritative checkout's unrelated changes.
- Merging the unrelated Dependabot PR merely because it is open.
- Weakening CI, dependency verification, authentication, authorization, production ACLs, or migration safety to accelerate closure.
- Treating unit/build output alone as local runtime evidence.

## Selected Delivery Design

Use seven sequential batches, each based on the latest merged `origin/main`: account security (#1258-#1264), public SEO/accessibility (#1265-#1272), social feeds (#1273-#1279), WFL (#1280-#1289), shared-folder integrity (#1290-#1298), command center (#1299-#1301), and build/supply-chain (#1302-#1307). Each batch receives a literal reviewed implementation plan, test-first implementation, alternate-port verification, a Builder test report, a focused PR, CI/CodeQL/Dependency Review, merge, production acceptance, and issue closure.

Alternatives were rejected: one 50-issue PR would be too risky to review, test, or roll back; 50 single-issue PRs would multiply CI/runtime/deployment overhead and create avoidable conflicts among changes to the same boundaries. Seven cohesive batches balance independence with shared implementation surfaces. The user's instruction to continue without routine approval is treated as approval of this delivery design and of plan corrections that do not materially expand scope.

## Cross-Cutting Requirements

### Compatibility and interfaces

- Retain Java 25, Spring Boot 4.1, Gradle Wrapper, MongoDB, Thymeleaf, vanilla ES modules, and repository-native response conventions.
- Prefer additive API evolution and safe defaults for existing Mongo documents; explicitly migrate data when uniqueness, indexing, retention, or edge collections change.
- Keep browser-cookie and explicit bearer-token clients working according to the existing supported contracts.
- Update owning package documentation and operator runbooks when persisted state, configuration, workflows, or recovery behavior changes.

### Security and privacy

- Fail closed at authentication, authorization, configuration, URL, persistence, and proxy-trust boundaries.
- Use stable public errors while retaining redacted internal diagnostics and audit categories.
- Bound and minimize retained personal, location, file, and operational data; document retention and cleanup.
- Do not trust client-supplied media duration, forwarding headers, URL schemes, ownership state, or concurrency assumptions without server-side validation.

### Data and concurrency

- Use atomic Mongo updates, uniquely indexed edge records, optimistic versions/CAS, bounded retries, or leases as appropriate; in-process locks alone are insufficient for cross-instance state.
- Versioned migrations must be idempotent, lease-protected, observable, and safe to retry after interruption.
- Pagination uses deterministic ordering, bounded page sizes, opaque validated cursors where requested, and stable continuation behavior.
- Cleanup preserves active/recoverable state and retains only bounded, redacted diagnostics.

### Testing and delivery

- Begin behavior changes with a failing behavioral test or recorded failing contract, then implement the smallest change that satisfies it.
- Add success, validation, authorization, concurrency, race, migration, retention, and failure-path coverage appropriate to each issue.
- Run focused tests first and `:website:check` before publication; run Pester/build verification when build or Windows operations change.
- Start the application on a non-8080 port and capture exact HTTP/UI inputs and responses for affected behavior.
- Do not merge before required CI, Dependency Review, and CodeQL gates pass. Do not close issues before merge and runtime evidence.

## Batch 1 - Account Security and Lifecycle

### Issues

#1258-#1264.

### Required behavior

- #1258: tie sessions/tokens to current account security state and revoke them after password reset, deletion, status, role, or permission changes.
- #1259: make unknown-account, wrong-password, and inactive-account login failures externally indistinguishable in status, body, and practical timing while retaining internal audit categories.
- #1260: store a versioned password-hash format with calibrated work factor and constant-time comparison; verify legacy hashes and upgrade them after successful login.
- #1261: return stable safe framework/client errors, keep parser details internal, and avoid ERROR stack traces for expected 4xx traffic while preserving unexpected-500 diagnostics.
- #1262: explicitly configure the production trusted-proxy chain, reject invalid CIDRs at startup, document the setting, ignore spoofed forwarding headers, and resolve the actual tunnel hop correctly.
- #1263: select one coherent account approval lifecycle, migrate drifted state, and enforce it through signup, approval, login, and admin status changes. Preserve the existing active-by-default product behavior unless current-source evidence shows production depends on manual approval; remove dormant approval state rather than introducing surprise signup blocking.
- #1264: remove JSON `consumes` from bodyless operations, return accurate creation/update/delete success statuses and headers, update API docs, and cover clients without `Content-Type`.

## Batch 2 - Public SEO, Metadata, and Accessibility

### Issues

#1265-#1272.

### Required behavior

- #1265: apply reusable `noindex,nofollow` metadata to private, admin, and authentication shells while leaving public content indexable.
- #1266: render a real 404 status with `noindex`; omit or safely use the requested canonical rather than canonicalizing missing pages to home.
- #1267: resolve public-safe data for profiles, posts, and restaurants in the view layer; render resource-specific title/description/canonical metadata; return 404 for missing or expired records.
- #1268: give top-rated WFL its own canonical and indexability while marking favorites non-indexable and excluding it from discovery.
- #1269: generate valid sitemap XML from an explicit public-route registry plus eligible dynamic content, excluding private/expired resources and supporting sitemap splitting at protocol limits.
- #1270: give both Bell archive pages one descriptive H1 inside a main landmark, logical headings, and `noopener noreferrer` on every new-tab link.
- #1271: require explicit button type for every static and dynamically rendered non-submit button, enforced by a static regression test.
- #1272: add standard field names and autocomplete tokens to login, signup, forgot-password, and reset-password fields without weakening validation or browser-session handling.

## Batch 3 - Social Relationships and Feed Scalability

### Issues

#1273-#1279.

### Required behavior

- #1273: store likes as atomic uniquely indexed relationships or an equivalent transaction-safe model, keep counts consistent, migrate embedded likes, and make retries idempotent.
- #1274: store follows as atomic uniquely indexed edges or equivalent atomic state, preserve profile counts and followed feeds, migrate arrays, and prevent duplicate/racy edges.
- #1275: obtain reply counts for a feed page in one bounded aggregation or transaction-safe counter path so query count remains constant as page size grows.
- #1276: apply visibility/expiration in queries or bounded refill logic so filtered records do not consume visible page capacity or break cursor continuation.
- #1277: deprecate or convert legacy account-post endpoints to bounded stable cursor pages and update callers compatibly.
- #1278: move expiration repair to an idempotent migration/maintenance path and prove feed GETs do not persist documents.
- #1279: maintain reply count atomically and update reply expirations with a bounded bulk/derived model, including large-thread and concurrent interaction coverage.

## Batch 4 - WFL Ownership, Concurrency, Retention, and Privacy

### Issues

#1280-#1289.

### Required behavior

- #1280: delete sessions owned by a deleted account, but remove/anonymize a non-owner participant and vote without destroying other members' sessions.
- #1281: enforce one documented participant cap across create, invite, and shared-link join paths with a stable full-session response under repeated and concurrent joins.
- #1282: make joins, votes, and restaurant changes atomic or optimistically locked with bounded retries and stable conflict responses.
- #1283: restrict restaurant resets to the creator/host, expose capability in responses/UI, audit resets, and reject stale sessions.
- #1284: define active lifetime/archive retention, prevent mutation after expiry, add index-backed cleanup, and preserve an honest historical UI state.
- #1285: hydrate up to 25 session summaries using one restaurant collection and one caller-specific rating/favorite batch rather than N+1 queries.
- #1286: paginate and stably sort admin restaurant inventory with bounded name/city/state filters and corresponding UI navigation.
- #1287: find duplicates with indexed Mongo aggregation, page previews, fetch only confirmed groups, and preserve version-match safety.
- #1288: normalize and accept only HTTP/HTTPS restaurant URLs at ingestion and rendering; build DOM links without unsafe interpolation.
- #1289: retain only minimal anonymous location state with explicit short expiry, owner/consent transitions, corrupt-data handling, and legacy-key migration.

## Batch 5 - Shared-Folder Integrity and Scalability

### Issues

#1290-#1298.

### Required behavior

- #1290: refresh the catalog asynchronously with explicit file/directory/depth/time/cancellation budgets, last-known-good fallback, and freshness/partial status.
- #1291: publish catalog generation/invalidation after every committed mutation and upload finalization so search/radio sees changes immediately and safely handles refresh failure.
- #1292: provide deterministic bounded search pages with opaque cursor, duplicate-name stability, authorization on every page, and usable results beyond 200 matches.
- #1293: wrap streaming downloads to audit completion, bytes, abort, and failure without buffering; deduplicate range/media noise.
- #1294: make radio transitions cross-instance safe with optimistic version/CAS or a station lease and bounded conflict retries.
- #1295: obtain duration from trusted server-side media metadata or tightly validate client reports; reject forged short/long values and handle unknown formats gracefully.
- #1296: remove completed upload sessions after a short history window through an indexed TTL or bounded purge without deleting active/recoverable sessions early.
- #1297: define terminal media-job retention and safely clean descriptors, partials, and reservations while keeping bounded redacted diagnostics and crash recovery.
- #1298: scope persisted resume state to a non-secret account identity or session storage, clear on logout/account change, verify ownership before rendering, and clean legacy global state.

## Batch 6 - Command-Center Configuration and Power Actions

### Issues

#1299-#1301.

### Required behavior

- #1299: validate paths, durations, thresholds, ports, byte limits, retry/action limits, and cross-field constraints at startup; fail production clearly while retaining safe simulated-development defaults.
- #1300: use one validated configured power delay for response deadlines and Windows arguments, observe launch/early process failure, and audit accepted versus failed actions without allowing arbitrary commands.
- #1301: persist minimal pending power-action state, reconcile it with OS/deadline state after restart, prevent overlapping actions, and make cancellation idempotent before and after the deadline.

## Batch 7 - Reproducible Builds, CI, and Supply Chain

### Issues

#1302-#1307.

### Required behavior

- #1302: derive release versions from explicit inputs and development versions from commit identity so the same commit resolves the same version across dates.
- #1303: remove mandatory network access from ordinary `processResources`; model the sensor bundle as pinned cached input or explicit packaging with offline support, timeouts, checksums, and declared inputs/outputs.
- #1304: run pinned Pester suites on Windows CI under PowerShell 7 and Windows PowerShell 5.1, upload NUnit XML on failure, and keep them out of non-Windows jobs.
- #1305: add PR/branch concurrency cancellation plus explicit job/critical-step timeouts sized for observed builds while preserving required independent main runs.
- #1306: pin every third-party action to a reviewed full SHA with readable version comments, configure controlled Dependabot updates, and reject mutable action references in repository checks.
- #1307: generate/review Gradle verification metadata, enforce strict verification in CI, document updates, and prove an untrusted artifact fails actionably.

## Expected Files and Ownership Boundaries

- Account/security/configuration packages, `application*.yml`, API docs, and matching tests for Batch 1.
- View controllers, templates, sitemap/public-route services, static JS, and accessibility/template tests for Batch 2.
- Post/account repositories, migrations, feed services, DTOs, and repository/service tests for Batch 3.
- WFL repositories/services/controllers/templates/JS, migrations, and browser/repository tests for Batch 4.
- Shared-folder catalog/search/streaming/radio/upload/media packages, static JS, configuration, migrations, and recovery tests for Batch 5.
- Command-center configuration/action persistence/Windows executor modules and tests for Batch 6.
- Root Gradle scripts, wrapper/config metadata, workflows, Dependabot configuration, production Pester tasks, documentation, and repository policy tests for Batch 7.

Exact files, line ranges, current code, proposed code, and commands belong in each batch implementation plan after refreshed-source inspection.

## Validation and Acceptance

A batch is accepted only when every included issue maps to code or documented already-satisfied evidence; focused and full required suites pass; affected behavior is exercised against a non-8080 local app with exact requests/responses; a validated Builder test report is pushed; the PR passes required gates and merges; production safely serves the merged behavior; and every issue receives closure evidence. The campaign closes only when `gh issue list --state open` returns no in-scope issues and Builder indexes, validation, closure, and session memory are pushed.

## Rollout and Recovery

- Execute batches in order unless refreshed-source evidence shows an explicit dependency requires a small reorder.
- Start each batch from the latest merged `origin/main`; do not stack unresolved branches across batches.
- Prefer additive migrations and compatibility reads before cleanup. Every migration/lease/TTL change must be idempotent and recoverable after interruption.
- Verify candidate runtime on a non-8080 port. Production changes use the existing deploy lock and native Windows service workflow.
- If production verification fails, keep or restore the previous release, leave affected issues open, and record the evidence; never weaken production ACLs to inspect protected metadata.

## Risks

- Edge migrations and concurrency changes can double-count or lose state without uniqueness and idempotency; migration and race tests are mandatory.
- Dynamic sitemap/metadata work can expose private or expired resources; eligibility rules must be centralized and tested.
- Shared-folder catalog and streaming changes affect filesystem and cross-instance behavior; budgets, cancellation, last-known-good state, and failure audits must be explicit.
- Strict dependency verification and SHA pinning can initially break builds until all plugin/tool artifacts are captured; verify on all supported platforms before merge.
- Seven substantial batches may conflict with Dependabot or new user work; refresh main before every batch and resolve only campaign-owned conflicts.

## Open Questions

None block execution. The issue bodies define acceptance intent; the active-by-default account lifecycle default preserves current product behavior unless direct production evidence requires manual approval. The user explicitly authorized autonomous continuation through implementation, testing, PR/CI, merge, closure, deployment verification, and Builder closeout without routine approval gates.
