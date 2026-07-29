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
- Batch 1 plan: [Account Security and Lifecycle Issues 1258-1264](../implementation-plans/2026-07-29-account-security-lifecycle-issues-1258-1264.md) (`ready-for-execution`).
- Batch 2 plan: [Public SEO and Accessibility Issues 1265-1272](../implementation-plans/2026-07-29-public-seo-accessibility-issues-1265-1272.md) (`ready-for-execution`).
- Later batch implementation plans will be linked after each current-source inspection and review.

## Spoke Repositories

- `christopherbell-dev`: authoritative checkout `A:\Projects\christopherbell.dev`; it is dirty, ahead 3, behind 90, and must remain untouched.
- Batch 1 worktree: `A:\Projects\christopherbell.dev-worktrees\all-open-issues-20260729` on `codex/all-open-issues-20260729`, created from refreshed `origin/main` commit `8405cd77d0f1743fe33d70cc80b47e37048090a0`; merged by PR #1319 as `e393687d10c40b856f35d669c25bf3ea65c5c083`.
- Batch 2 worktree: `A:\Projects\christopherbell.dev-worktrees\issues-1265-1272-20260729` on `codex/issues-1265-1272-20260729`, created from refreshed `origin/main` merge commit `e393687d10c40b856f35d669c25bf3ea65c5c083`.

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
- The user explicitly authorized autonomous continuation without routine approval pauses.

## Blockers

None.

## Validation

- Current issue bodies, author, comment counts, open pull requests, remotes, branches, worktrees, and authoritative-checkout status were inspected directly on 2026-07-29.
- Each batch must pass focused tests, full relevant regression coverage, alternate-port runtime acceptance, required GitHub checks, merge confirmation, issue closure, and production-safe verification before campaign closeout.
- Batch 1 runtime fixtures, alternate listeners, and disposable databases were removed. Production acceptance additionally confirmed the generic 401 login envelope and that a bodyless DELETE reaches authorization (403) rather than failing with 415.

## Next Steps

1. Execute the reviewed Batch 2 plan test-first in its isolated worktree.
2. Publish, independently review, CI-validate, merge, production-verify, and close #1265-#1272.
3. Implement, test, publish, merge, close, and production-verify the remaining batches in dependency order.
4. Refresh Builder indexes, close this record, and save final campaign memory after all 50 issues are closed.
