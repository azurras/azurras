# 2026-07-28 Void Keep Alive Relaunch

## 20:56 - Release 1 completed and deployed

### Request

Implement Release 1 of the approved three-release Void public-growth program without repeated approval stops. Make the survival proposition obvious, present the existing Like mechanic as Keep alive, remove popularity sorting, improve sharing, add safe post previews, and disclose no content for missing or expired posts. Preserve the existing expiration and authorization domain, prioritize working behavior and security, commit and push completed work, merge it, allow the automatic deployment, and verify production before starting Release 2.

### Project Context

- Builder hub: `C:\Users\Christopher\Developer\builder`, branch `main`.
- Spoke: `A:\Projects\christopherbell.dev`.
- The authoritative spoke checkout contained user changes and was preserved untouched.
- Implementation used isolated worktree `A:\Projects\christopherbell.dev-worktrees\void-keep-alive-relaunch` on branch `codex/void-keep-alive-relaunch` from main SHA `7d7d042c26d7bbabee2cdf0bc430127a0020e65e`.
- The Like API and post-expiration services remained compatibility and authority boundaries; only the public vocabulary and rendering changed.
- The user has approved continued execution of Releases 2 and 3 without approval checkpoints unless new authority, material scope change, or an unresolved external blocker appears.

### Work Completed

- `/void` now leads with `Nothing lasts unless people care` and explains the 24-hour root, keep-alive, and reply rules in the first viewport.
- Visible Like presentation became `Keep alive · +24h` / `Kept alive`, with a quiet count, accessible effect label, and UI updates only after the server response.
- Added native Share with clipboard/prompt fallback while retaining Copy link.
- Removed the engagement-derived Active sort; Newest and Expiring Soon remain.
- Corrected thread/reply lifespan copy so the whole-thread behavior is explicit.
- Added `VoidPostSocialPreview` and its service for normalized, Unicode-bounded active-post metadata.
- Active post metadata is Thymeleaf-escaped and no-store. Missing or expired `/p/{id}` responses return a content-free no-store 404 page and catch only the domain not-found exception.
- Added JavaScript, MVC, and Java tests for keep-alive state, sharing outcomes, ordering, Unicode bounds, escaping, and vanished responses.
- Spoke commits `ba127e8d` and `ba3d2194` were pushed, PR `azurras/christopherbell.dev#1314` passed required checks, and squash merge `7e958e737b34563d6d49a078243437d5fa9e3377` reached production automatically.
- The program spec now marks Release 1 complete and Release 2 as next.

### Decisions

- Kept backend Like route names unchanged to avoid breaking current clients.
- Made server-returned `liked`, `likesCount`, and `expiresOn` authoritative; there is no optimistic lifespan invention.
- Used time-only ordering and did not expose an engagement-ranked replacement.
- Used the active-domain lookup before generating metadata and mapped only `ResourceNotFoundException` to the vanished page.
- Retained the existing generic 1200x630 preview image for this release.

### Validation

- Task 1 RED: new Keep alive presentation and feed-sort imports failed before implementation. Focused JavaScript tests then passed 26 of 26 and the focused view test passed.
- Task 2 RED: preview types did not compile before implementation. Five focused Java tests passed after implementation.
- Additional RED proved active and missing post pages lacked no-store headers; both passed after controller caching was corrected.
- Final pre-merge command: `$env:GRADLE_USER_HOME='A:\Temp\gradle-void-relaunch'; .\gradlew.bat :website:check --no-daemon` -> `BUILD SUCCESSFUL in 1m 34s`.
- Local desktop/mobile browser checks found no horizontal overflow, showed the rules in the first viewport, and exposed the expected accessible Keep alive controls.
- Corrected isolated runtime on port 8081 proved account creation/login, post creation, exact 24-hour Keep alive extension, active metadata, content-free 404, and no-store headers.
- PR checks passed on Windows, Linux, and macOS, plus dependency review and all CodeQL languages.
- Post-merge CI run `30414906393` and CodeQL run `30414906457` passed.
- Production served the exact merge SHA and new Void copy. `/`, `/void`, `/messages`, `/music`, and `/back-office` returned HTTP 200; missing `/p/{id}` returned generic HTTP 404 with no-store and no identifier leak.
- Production services `ChristopherBellDev`, `ChristopherBellMediaWorker`, `cloudflared`, and `MongoDB` were Running and Automatic after deployment.
- Durable test report: `docs/test-reports/2026-07-28-void-keep-alive-relaunch.md`.

### Runtime Isolation Incident and Cleanup

The first local runtime set only `SPRING_MONGODB_URI`. Because the local profile independently fixes `spring.mongodb.database` to `christopherbell`, it created one test account, one browser session, and one post in the production database. Production verification exposed the fixture immediately.

A read-only cross-collection query found only these exact documents:

- Post `a4f8ef3a-c430-4c1d-b6ce-fc771e2a2cc9`.
- Browser session `3cd85a6c-4747-492a-ace8-7128bb97d0f0`.
- Account `2e7d40df-5a8d-47dd-be34-3c120cb75e3b` (`void_f032bef7`).

All three were deleted with exact identifiers and identifying fields. Follow-up counts were zero, and the public feed no longer contained the post. The runtime was repeated with both `SPRING_MONGODB_URI=mongodb://127.0.0.1:27017` and `SPRING_MONGODB_DATABASE=christopherbell_void_relaunch_test`; corrected fixture IDs existed only in the isolated database and had zero matches in production. Future local verification must set both properties.

### Closure Readiness

ready

### Closure Text

Release 1 of the user-approved Void public-growth program is complete. PR #1314 merged as `7e958e737b34563d6d49a078243437d5fa9e3377` after Windows, Linux, macOS, dependency-review, and CodeQL checks passed. Full local verification and a corrected isolated runtime proved the Keep alive, sharing, ordering, preview, and vanished-post contracts. The automatic deployment reached the exact merge SHA, production smoke checks passed, and the temporary test-data isolation incident was fully remediated and documented. There was no separate source GitHub issue and no GitHub comments or attachments were used as closure instructions. Continue with Release 2 - Public Discovery under the approved program spec.

### Current State

- Spoke PR: merged.
- Production: merge `7e958e737b34563d6d49a078243437d5fa9e3377` live.
- Local corrected candidate: stopped; port 8081 closed.
- Production services: healthy.
- Isolated worktree retained for PR provenance; the authoritative dirty spoke checkout remains untouched.
- Builder plan `docs/implementation-plans/2026-07-28-christopherbell-dev-void-keep-alive-relaunch.md`: complete.
- Program spec `docs/specs/2026-07-28-void-public-growth-program.md`: still ready for execution because Releases 2 and 3 remain.

### Follow-ups

- Begin Release 2 with a focused implementation plan for Public Discovery.
- Preserve non-popularity ordering and the explicit `lastExtendedOn` semantics from the approved spec.
- Always override both Mongo URI and database for future local candidate runtimes.
- One brief HTTP 502 occurred during the expected production listener rotation; the target release then served HTTP 200. Treat zero-downtime deployment as separate scope if desired.
