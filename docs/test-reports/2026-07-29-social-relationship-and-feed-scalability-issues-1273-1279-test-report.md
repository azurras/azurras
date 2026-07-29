# Social Relationship and Feed Scalability Issues 1273-1279 Test Report

## Document Status

complete

## Story/Issue

GitHub issues `azurras/christopherbell.dev` #1273 through #1279.

## Branch

`codex/issues-1273-1279-rebased-20260729`, source commit `b6d7835e6ee0975b19d58ed048b39c65912b9aa8`, based on `a6a88e91f35bcbf9eeadeaf06cbf93df80ce0a5f` and squash-merged as `e3afbf3c9eeb65525f573f299f82287ef8665554`.

## App / Environment

The exact website JAR ran with profile `local` at `http://127.0.0.1:8093` against disposable database `cbell_issue_1273_1279_20260729`. Both Mongo URI and database were explicitly set. Production port 8080 and its database were not changed during local acceptance.

## Local Run Details

The final JAR ran hidden as PID `55084`. Fixtures included duplicate legacy like/follow arrays, 120 expired posts ahead of active rows, 150 account-history posts, and a runtime account created through the real CSRF signup flow. The exact PID was stopped, port 8093 was confirmed free, and the exact database was dropped and confirmed absent.

## Test Cases

| Case | Expected | Result |
|---|---|---|
| Full gate | Java, JavaScript, packaging, sensor, policy pass | Pass |
| V009/V010 | Deduplicate relationships, remove arrays, backfill metrics/expiration | Pass |
| Desired-state retries | Repeated PUT/DELETE is idempotent | Pass |
| Concurrent relationships | Twenty concurrent operations retain one authoritative state | Pass |
| Feed capacity | Expired rows do not consume stable or legacy limits | Pass |
| Read purity | Repeated feed/thread GETs cause no persistence changes | Pass |
| Account histories | Stable cursor page and legacy 100-row cap/deprecation | Pass |
| Cleanup | Exact process, listener, and database removed | Pass |

## Data Sent

- Seeded legacy `likedBy` and `followingIds` arrays with duplicate values and one root/reply thread.
- Seeded 120 newer expired rows before the active runtime root/reply and requested both stable and legacy feeds with size 2.
- Sent sequential retry and twenty-way concurrent `PUT /api/posts/2026-07-29/{postId}/like`, `DELETE /api/posts/2026-07-29/{postId}/like`, `POST /api/accounts/2025-09-14/{accountId}/follow`, and matching `DELETE` requests.
- Sent 50 `GET /api/posts/2026-07-29/feed` and thread GET cycles and compared root/post/follow persistence before and after.
- Seeded 150 account-history posts and requested `GET /api/posts/2026-07-29/account/{accountId}?size=1000` and the legacy history route.

## Response Received

The running app returned status code: 200 for the root, feed, thread, history, and every relationship request. Migrations 001-010 were recorded. V009 produced one deterministic like edge and one follow edge from the legacy fixtures, removed both arrays, and V010 set reply counts and matching root/reply expiration. Both feed responses contained the two active posts despite 120 newer expired rows. Sequential like/follow counts were `2,2,1,1`; all concurrent responses had status code: 200; final edge/counter state was singular and correct. Fifty read cycles left timestamps, expirations, counters, and relationship counts unchanged. Stable history returned status code: 200 with 100 rows plus a cursor; legacy returned status code: 200 with 100 rows, `Deprecation: true`, and a successor `Link`.

## Pass / Fail

All accepted cases passed. Final `:website:check` completed 1,431 Java tests with 0 failures/errors and 3 skipped, followed by successful JavaScript, boot JAR, sensor-runtime, and policy checks. `git diff --check` passed.

## Evidence

- Pull request [#1323](https://github.com/azurras/christopherbell.dev/pull/1323) passed Ubuntu, macOS, Windows, dependency review, and all CodeQL analyses.
- Post-merge main CI and CodeQL passed for `e3afbf3c9eeb65525f573f299f82287ef8665554`.
- Production PID 60136 started after merge; local/public root contained the exact merge SHA and liveness/readiness returned 200 `UP`.
- Production migrations 009 and 010 are `APPLIED`; required edge indexes exist; legacy arrays and missing root metrics each count zero.

## Bugs / Follow-ups

No unresolved Batch 3 defect remains. Continue with WFL issues #1280-#1289.
