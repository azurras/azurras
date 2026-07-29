# Complete christopherbell.dev Issues 1258-1307

- Status: active
- Owner/Agent: Codex primary agent
- Started: 2026-07-29

## Objective

Resolve every currently open issue in `azurras/christopherbell.dev` through current-state validation, implementation, alternate-port local runtime testing, pull requests, required CI, merge, issue closure, production-safe verification, and Builder closeout.

## Scope

- 50 open issues: [#1258](https://github.com/azurras/christopherbell.dev/issues/1258) through [#1307](https://github.com/azurras/christopherbell.dev/issues/1307).
- Only comments authored by `azurras` may change scope or acceptance intent; all 50 issues currently have zero comments and no attached instructions.
- The unrelated Dependabot PR #1310 is outside this campaign unless it becomes a direct dependency or conflict.

## Related Specs and Plans

- Project spec: [Complete christopherbell.dev Issues 1258-1307](../specs/2026-07-29-complete-christopherbell-dev-issues-1258-1307.md) (`ready-for-execution`).
- Batch 5 spec: [Shared-Folder Integrity and Retention Issues 1290-1297](../specs/2026-07-29-shared-folder-integrity-retention-issues-1290-1297.md) (`ready-for-execution`).
- Batch 1 plan: [Account Security and Lifecycle Issues 1258-1264](../implementation-plans/2026-07-29-account-security-lifecycle-issues-1258-1264.md) (`ready-for-execution`).
- Batch 2 plan: [Public SEO and Accessibility Issues 1265-1272](../implementation-plans/2026-07-29-public-seo-accessibility-issues-1265-1272.md) (`complete`).
- Batch 3 plan: [Social Relationship and Feed Scalability Issues 1273-1279](../implementation-plans/2026-07-29-social-relationship-and-feed-scalability-issues-1273-1279.md) (`complete`).
- Batch 4 plan: [WFL Session and Restaurant Safety/Scalability Issues 1280-1289](../implementation-plans/2026-07-29-wfl-session-restaurant-safety-scalability-issues-1280-1289.md) (`complete`).
- Batch 5 plan: [Shared-Folder Integrity and Retention Issues 1290-1297](../implementation-plans/2026-07-29-shared-folder-integrity-retention-issues-1290-1297.md) (`ready-for-execution`).
- Later batch implementation plans will be linked after each current-source inspection and review.

## Spoke Repositories

- `christopherbell-dev`: authoritative checkout `A:\Projects\christopherbell.dev`; it is dirty, ahead 3, behind 90, and must remain untouched.
- Batch 1 worktree: `A:\Projects\christopherbell.dev-worktrees\all-open-issues-20260729` on `codex/all-open-issues-20260729`, created from refreshed `origin/main` commit `8405cd77d0f1743fe33d70cc80b47e37048090a0`; merged by PR #1319 as `e393687d10c40b856f35d669c25bf3ea65c5c083`.
- Batch 2 worktree: `A:\Projects\christopherbell.dev-worktrees\issues-1265-1272-20260729` on `codex/issues-1265-1272-20260729`; rebased onto refreshed `origin/main` commit `5de2a8b02941ff7e95b6f2648b7bada9397f68b9` and merged by PR #1321 as `f31535f29312d24573a6031b0162aa8ebc4b5318`.
- Batch 3 worktree: `A:\Projects\christopherbell.dev-worktrees\issues-1273-1279-rebased-20260729` on `codex/issues-1273-1279-rebased-20260729`; based on refreshed `origin/main` commit `a6a88e91f35bcbf9eeadeaf06cbf93df80ce0a5f` and merged by PR #1323 as `e3afbf3c9eeb65525f573f299f82287ef8665554`.
- Batch 4 worktree: `A:\Projects\christopherbell.dev-worktrees\issues-1280-1289-20260729` on `codex/issues-1280-1289-20260729`; rebased onto refreshed `origin/main` commit `e3f7c676e8bf73a11056b9f009723ba9628025e8` and merged by PR #1325 as `b28031d535effef1fcbd547ba8f7dffdd4e76193`.
- Batch 5 worktree: `A:\Projects\christopherbell.dev-worktrees\issues-1290-1297-20260729` on `codex/issues-1290-1297-20260729`, based on refreshed `origin/main` commit `b28031d535effef1fcbd547ba8f7dffdd4e76193`.

## Dispatched Tasks

- No external agent tasks dispatched. The primary agent is executing the campaign.

## Current State

- GitHub inventory on 2026-07-29 confirms exactly 50 open issues, #1258-#1307, and one unrelated open Dependabot pull request.
- Every issue was authored by `azurras`, has a non-empty evidence-backed body, and has zero comments.
- The campaign is divided into seven dependency-aware batches: account security; public SEO/accessibility; social feeds; WFL; shared-folder integrity; command center; and build/supply-chain.
- The full clean-mainline `:website:check` baseline passed in 3m44s with Java, JavaScript, packaged JAR, and sensor-runtime verification.
- Batch 1 current-source inspection found that browser sessions already re-check an account security fingerprint; the reviewed design centralizes that invariant and extends it to bearer JWTs.
- Batch 1 implementation for #1258-#1264 is merged. Final `:website:check` passed with 1,393 Java tests, 269 JavaScript tests, packaged-JAR and sensor-runtime verification; `:cbell-lib:test` separately passed 101 tests.
- Batch 1 alternate-port acceptance on port 8093 passed against disposable Mongo databases, including `201 + Location`, current password storage, uniform failed login, immediate stale-token rejection, legacy timing parity, concurrent credential upgrade, moderation-safe atomic login, stale JWT browser-exchange rejection, safe malformed JSON, bodyless DELETE, and cookie-mode login. Evidence: [Batch 1 test report](../test-reports/2026-07-29-account-security-and-lifecycle-issues-1258-1264-test-report.md).
- Independent review found and drove fixes for credential timing, null legacy salt handling, deterministic concurrent upgrades, whole-document login lost updates, and stale JWT-to-browser-session exchange. The final `fc294f7d..9be7ef2c` review reported no remaining Critical or Important blocker.
- PR [#1319](https://github.com/azurras/christopherbell.dev/pull/1319) passed Ubuntu, macOS, Windows, all CodeQL languages, and dependency review, then squash-merged as `e393687d10c40b856f35d669c25bf3ea65c5c083`.
- Production rotated from PID 39760 to PID 48484 and serves asset URLs containing the merge SHA. Local root, readiness, and public root return 200; V008 is APPLIED with the expected checksum; all 20 accounts remain; no retired approval field or test account remains; MongoDB, ChristopherBellDev, and cloudflared are Running/Automatic.
- Issues #1258-#1264 are closed with merge, CI, runtime, production, test-report, and session-memory evidence; 43 campaign issues remain open.
- Batch 2 source inspection confirmed post metadata/expiration-aware 404 handling is partly present, while profile and restaurant views remain generic. The reviewed design centralizes view indexing metadata, makes missing dynamic resources true non-indexable 404s, and replaces the static sitemap with an explicit route/data registry and 50,000-URL shard contract.
- Batch 2 implemented centralized indexing metadata, true unknown/dynamic 404s, entity-aware public metadata, bounded generated sitemap data, WFL canonical policy, The Bell semantics, explicit button behavior, and password-manager form metadata. Final `:website:check` passed 1,418 Java tests with zero failures/errors and 3 skipped plus all frontend/package/runtime policy gates.
- PR [#1321](https://github.com/azurras/christopherbell.dev/pull/1321) passed Ubuntu, macOS, Windows, CodeQL, and dependency review and squash-merged as `f31535f29312d24573a6031b0162aa8ebc4b5318`. Production rotated from PID 51060 to PID 46940, served merge-SHA assets, and passed liveness, readiness, local/public root, sitemap, noindex 404, protected namespace, canonical, The Bell, and auth-form acceptance.
- Issues #1265-#1272 are closed with merge, CI, runtime, production, and test-report evidence; 35 campaign issues remain open.
- Batch 3 moved likes and follows into deterministic unique edge collections; added retry-safe desired-state like/follow operations; assembled feed engagement in constant query counts; filtered visibility before page limits; bounded legacy histories; and removed expiration repair writes from reads. Final `:website:check` passed 1,431 Java tests with zero failures/errors and 3 skipped plus frontend, package, sensor, and policy gates.
- PR [#1323](https://github.com/azurras/christopherbell.dev/pull/1323) passed Ubuntu, macOS, Windows, CodeQL, and dependency review and squash-merged as `e3afbf3c9eeb65525f573f299f82287ef8665554`. Production rotated to PID 60136, served merge-SHA assets locally and publicly, reported liveness/readiness `UP`, applied migrations 009/010, created the required unique edge indexes, removed all legacy relationship arrays, and backfilled every root metric.
- Issues #1273-#1279 are closed with merge, CI, concurrency/runtime, production migration, and test-report evidence; 28 campaign issues remain open.
- Batch 4 intake confirmed #1280-#1289 are open, authored by `azurras`, and have no comments or attachments. Current-source inspection found destructive shared-session deletion, unbounded/non-atomic membership and mutation state, participant-authorized resets, no lifecycle/TTL, list N+1 hydration, unbounded inventory/dedupe reads, unsafe interpolated website schemes, and privacy-heavy persistent anonymous browser state. The reviewed design uses one-document atomic session mutations, bounded lifecycles/audit, indexed page queries, defense-in-depth URL validation, and expiring ID-only browser state.
- Batch 4 implemented participant-safe account deletion; atomic capped and revisioned WFL mutations; creator-only resets; bounded lifecycle, audit, and TTL; constant-query list hydration; indexed inventory and duplicate cursors; safe website validation; and anonymous ID-only storage without ZIP or coordinates. Final `:website:check` passed 1,489 Java tests with zero failures/errors and 3 skipped; a direct browser run passed 288/288.
- PR [#1325](https://github.com/azurras/christopherbell.dev/pull/1325) passed Ubuntu, macOS, Windows, CodeQL, and dependency review and squash-merged as `b28031d535effef1fcbd547ba8f7dffdd4e76193`. Production rotated from PID 48420 to PID 52804, served the exact normalized merged client asset, reported liveness/readiness 200, applied migration 011 with the expected checksum/indexes, and retained zero unsafe websites or missing lifecycle fields.
- Issues #1280-#1289 are closed with merge, CI, runtime, production migration, and test-report evidence. Issue #1298 was already closed by PR #1324, leaving 17 campaign issues open: #1290-#1297 and #1299-#1307.
- Batch 5 design chooses asynchronous bounded immutable catalog generations with last-known-good fallback, generation-bound search cursors, post-commit invalidation, stream-terminal auditing, optimistic radio CAS with trusted Music metadata, and cleanup-before-TTL retention.
- Batch 5 implementation plan passed mechanical validation and execution-readiness review with seven ordered task-specific Code Edit blocks, mandatory code-skill invocation, concrete red/green commands, runtime acceptance, rollback, risks, and completion criteria. No blocker remains.
- The user explicitly authorized autonomous continuation without routine approval pauses.

## Blockers

None.

## Validation

- Current issue bodies, author, comment counts, open pull requests, remotes, branches, worktrees, and authoritative-checkout status were inspected directly on 2026-07-29.
- Each batch must pass focused tests, full relevant regression coverage, alternate-port runtime acceptance, required GitHub checks, merge confirmation, issue closure, and production-safe verification before campaign closeout.
- Batch 1 runtime fixtures, alternate listeners, and disposable databases were removed. Production acceptance additionally confirmed the generic 401 login envelope and that a bodyless DELETE reaches authorization (403) rather than failing with 415.

## Next Steps

1. Inspect refreshed merged main and save/review the Batch 5 shared-folder plan for #1290-#1297.
2. Test, publish, merge, close, and production-verify the remaining batches in dependency order.
3. Refresh Builder indexes, close this record, and save final campaign memory after all 50 issues are closed.
