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
- Later batch implementation plans will be linked after each current-source inspection and review.

## Spoke Repositories

- `christopherbell-dev`: authoritative checkout `A:\Projects\christopherbell.dev`; it is dirty, ahead 3, behind 90, and must remain untouched.
- Campaign worktree: `A:\Projects\christopherbell.dev-worktrees\all-open-issues-20260729` on `codex/all-open-issues-20260729`, created from refreshed `origin/main` commit `8405cd77d0f1743fe33d70cc80b47e37048090a0`.

## Dispatched Tasks

- No external agent tasks dispatched. The primary agent is executing the campaign.

## Current State

- GitHub inventory on 2026-07-29 confirms exactly 50 open issues, #1258-#1307, and one unrelated open Dependabot pull request.
- Every issue was authored by `azurras`, has a non-empty evidence-backed body, and has zero comments.
- The campaign is divided into seven dependency-aware batches: account security; public SEO/accessibility; social feeds; WFL; shared-folder integrity; command center; and build/supply-chain.
- The full clean-mainline `:website:check` baseline passed in 3m44s with Java, JavaScript, packaged JAR, and sensor-runtime verification.
- Batch 1 current-source inspection found that browser sessions already re-check an account security fingerprint; the reviewed design centralizes that invariant and extends it to bearer JWTs.
- Batch 1 implementation for #1258-#1264 is complete in the isolated spoke worktree. Final `:website:check` passed with 1,389 Java tests, 269 JavaScript tests, packaged-JAR and sensor-runtime verification; `:cbell-lib:test` separately passed 100 tests.
- Batch 1 alternate-port acceptance on port 8093 passed against a disposable Mongo database, including `201 + Location`, current password storage, uniform failed login, immediate stale-token rejection, safe malformed JSON, and bodyless DELETE. Evidence: [Batch 1 test report](../test-reports/2026-07-29-account-security-and-lifecycle-issues-1258-1264-test-report.md).
- The user explicitly authorized autonomous continuation without routine approval pauses.

## Blockers

None.

## Validation

- Current issue bodies, author, comment counts, open pull requests, remotes, branches, worktrees, and authoritative-checkout status were inspected directly on 2026-07-29.
- Each batch must pass focused tests, full relevant regression coverage, alternate-port runtime acceptance, required GitHub checks, merge confirmation, issue closure, and production-safe verification before campaign closeout.
- Batch 1 runtime fixtures, alternate listener, and disposable database were removed; production remained running and both local and public roots returned 200.

## Next Steps

1. Commit Batch 1 spoke changes, publish its pull request, pass required CI, merge, close #1258-#1264, and verify production.
2. Inspect current source and save one literal, reviewed implementation plan for Batch 2.
3. Implement, test, publish, merge, close, and production-verify the remaining batches in dependency order.
4. Refresh Builder indexes, close this record, and save session memory after all 50 issues are closed.
