# 2026-07-28 - ChristopherBell.dev Void Public Discovery Release

## 22:10 - ChristopherBell.dev Void Public Discovery Release

### Request

Complete the second of the three approved Void public-growth releases without repeated approval pauses. The release had to make discovery useful without popularity ranking, preserve media playback/navigation behavior, remain safe for anonymous readers, use one PR, commit and push after each task, pass security-focused validation, merge, deploy automatically from `main`, and receive exact-SHA production verification.

### Project Context

- Builder hub: `C:\Users\Christopher\Developer\builder`, branch `main`.
- Spoke: `azurras/christopherbell.dev`.
- Implementation worktree: `A:\Projects\christopherbell.dev-worktrees\void-public-discovery`, branch `codex/void-public-discovery`.
- The authoritative checkout at `A:\Projects\christopherbell.dev` had unrelated user state and was not modified.
- Production runs natively on Windows. Port 8080 is production; all pre-merge runtime checks used port 8081 and an explicitly separate Mongo database.
- The user has approved autonomous continuation through the three-release program. Ask only for genuinely new authority, a material scope change, or an unresolved external blocker.

### Work Completed

Release 2 shipped through PR `azurras/christopherbell.dev#1315` and production SHA `f77c5f5bb644cc75cf98b27e722efdc00cd036f1`.

- Added normalized, deduplicated, bounded post topics and explicit root `lastExtendedOn` timestamps.
- Updated genuine revival behavior so confirmed keep-alives and replies move the timestamp while undo and unrelated actions do not.
- Added bounded, cursor-based, no-store New, Fading, Revived, Topics, topic, and People discovery APIs.
- Added privacy-aware account suggestions: signed-in overlap ranking and anonymous UTC-day rotation, excluding self, existing follows, mute, block in either direction, missing accounts, and suspended accounts.
- Added stricter fixed-window budgets for accounts younger than seven days: 10 roots, 30 replies, 60 added keep-alives, and 30 new follows per hour. Undo/unfollow and duplicate no-op mutations do not consume the add budget.
- Extended login tokens to seven days.
- Added public `/void/explore` and `/void/topic/{topic}` pages, a top-level Explore nav destination, five independently loaded panels, section-local retry/empty/load-more behavior, safe topic/person rendering, and responsive Void styling.
- Added Mongo migrations 004 and 005 with eight named indexes supporting discovery and trust exclusion queries.
- During full runtime verification, found that the two hand-written discovery `@Repository` classes were `final`, which prevented Spring exception-translation proxying. Added `VoidDiscoveryRepositoryProxyCompatibilityTest`, witnessed it fail, made both repositories proxyable, and witnessed it pass before repeating complete runtime and automated checks.
- Saved the local runtime report at `docs/test-reports/2026-07-28-christopherbell-dev-void-public-discovery-test-report.md`.
- Updated the Release 2 implementation plan to complete and advanced the program spec to Release 3.

### Decisions

- Kept chronological/time-based and topic-overlap ordering. Likes, keep-alives, replies, follower totals, and calculated lifespan never rank discovery.
- Kept responses `Cache-Control: no-store`, active-only, cursor-bounded, and public-safe.
- Reused the existing top-document media/navigation owner so normal Explore and topic links do not replace an active media element.
- Used one coherent release PR. GitHub rejected a merge-commit attempt under the active branch policy; the PR was merged by rebase, preserving the task commits.
- Did not weaken production ACLs when `prod.cmd status` and `auto-status` were denied. Verified automatic deployment through the public release SHA, listener rotation, service state, endpoints, and database migration/index state.

### Validation

- Final local full check: `$env:GRADLE_USER_HOME='A:\Temp\gradle-void-discovery'; .\gradlew.bat :website:check --no-daemon --console=plain` â€” BUILD SUCCESSFUL in 1m 36s.
- Final test evidence: 1,304 Java tests, zero failures/errors, three skipped; 265 JavaScript tests, zero failures/errors; sensor-runtime check passed.
- Isolated runtime: port 8081, PID 46824, database `christopherbell_void_discovery_20260728_01`; all six discovery APIs returned 200 and `no-store`.
- Browser runtime: five populated Explore panels, section isolation, New pagination from 12 to 24 cards, `/void/topic/music` with six cards, and desktop visual screenshot.
- Fixture safety: production documents matching `_id: /^e2e-void-/` were zero before and after; the isolated database was dropped; port 8081 was closed.
- PR checks: Ubuntu, macOS, Windows, Dependency Review, CodeQL Actions, CodeQL Java/Kotlin, and CodeQL JavaScript/TypeScript all passed.
- Automatic production deployment rotated from `7e958e737b34563d6d49a078243437d5fa9e3377` to exact SHA `f77c5f5bb644cc75cf98b27e722efdc00cd036f1` without visible prompts or UAC interaction.
- Production: local port 8080, apex `/void/explore`, and `www` `/void/explore` all returned 200; discovery New/Topics/People returned 200 and `no-store`; the live browser rendered real New/Fading/People results and correct Revived/Topics empty states.
- Production migrations 004/005 and all eight `void_*` indexes were present. `ChristopherBellDev` was Running with Java listener PID 34768 and wrapper PID 33148.

### Current State

- PR #1315 is merged.
- Remote `main` and production serve `f77c5f5bb644cc75cf98b27e722efdc00cd036f1`.
- The feature worktree remains registered locally; the remote feature branch deletion step could not switch the worktree to `main`, but this did not affect the merge or deployment.
- Builder `main` already contains runtime report commit `5b7a73b`; this memory/spec/plan closeout will be committed separately.
- Production port 8080 is healthy. Temporary port 8081 and the isolated test database are absent.

### Follow-ups

Release 3, ActivityPub Federation, is the next approved release in `docs/specs/2026-07-28-void-public-growth-program.md`. It is security-sensitive and must remain staged behind separate inbound/outbound flags. Start with fresh `origin/main`, a new isolated worktree, and a concrete Release 3 implementation plan. The approved rollout gates are discovery/metadata first, outbound delivery to a controlled peer, opted-in outbound production, inbound follows, and only then signed/idempotent inbound keep-alives and replies after SSRF, replay, rate-limit, moderation, and lifespan evidence passes.
