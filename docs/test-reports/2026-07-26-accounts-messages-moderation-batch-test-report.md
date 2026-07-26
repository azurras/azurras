# Accounts, Messages, Notifications, Posts, Reports, and Moderation Batch Test Report

## Document Status

complete

## Story/Issue

GitHub issues `cbell504/website#1155`, `#1156`, and `#1158` through `#1168`.

## Branch

- Repository: `A:\Projects\christopherbell.dev-worktrees\accounts-messages-moderation-1155-1168`
- Branch: `codex/accounts-messages-moderation-1155-1168`
- Commit under test: `349ba1c77dd33f2a077750600c7ff3086b31a7b0`

## App / Environment

- App: `christopherbell.dev` Spring Boot website
- Profile: `local`
- Base URL: `http://127.0.0.1:8090`
- Production URL/port preserved during testing: `http://127.0.0.1:8080`
- Isolated MongoDB database: `codex_batch5_final_20260726`
- MongoDB URI: `mongodb://127.0.0.1:27017`
- Mail delivery: disabled with `APP_MAIL_ENABLED=false`
- Gradle home: `A:\Projects\.gradle-codex-batch5`

## Local Run Details

The app was started from the isolated feature worktree with these explicit settings:

```powershell
$env:SERVER_PORT='8090'
$env:SPRING_PROFILES_ACTIVE='local'
$env:SPRING_MONGODB_URI='mongodb://127.0.0.1:27017'
$env:SPRING_MONGODB_DATABASE='codex_batch5_final_20260726'
$env:APP_MAIL_ENABLED='false'
$env:GRADLE_USER_HOME='A:\Projects\.gradle-codex-batch5'
.\gradlew.bat :website:bootRun --no-daemon
```

The final alternate-port application listened as Java PID `56928`. Standard output was captured in
`build/local-final-8090.log` and standard error in `build/local-final-8090.err.log` inside the
worktree. PID `56928` was stopped after testing and port `8090` was confirmed free. Production
Java PID `47288` remained listening on port `8080` throughout the test.

## Test Cases

| Case | Behavior | Result |
| --- | --- | --- |
| 1 | Public home renders from the candidate build | PASS |
| 2 | Back Office public shell includes the audit filter, pagination, and page script mounts | PASS |
| 3 | Public stable post feed returns the cursor-page response contract | PASS |
| 4 | Malformed stable post cursor is rejected | PASS |
| 5 | Admin account search rejects an anonymous caller | PASS |
| 6 | Conversation page API rejects an anonymous caller | PASS |
| 7 | Notification page API rejects an anonymous caller | PASS |
| 8 | Report page API rejects an anonymous caller | PASS |
| 9 | Moderation audit page API rejects an anonymous caller | PASS |
| 10 | Report resolution rejects an anonymous caller before accepting the supplied mutation | PASS |
| 11 | Legacy and additive account-deletion routes both preserve their protected boundary | PASS |
| 12 | Legacy and additive report-submission routes both preserve their protected boundary | PASS |
| 13 | Deletion removes retained like/edit identity and uses bounded private-state pages | PASS |
| 14 | Moderation audit retries are durable, idempotent, redact sensitive reason content, and preserve the original actor across a different-admin retry | PASS |
| 15 | Worker cancellation timeout preserves private media artifacts for retry | PASS |
| 16 | Failed notification persistence releases dedupe state, refunds rate capacity, and expired claims can retry immediately | PASS |
| 17 | Post edits remain original-author-only, including for administrators | PASS |

## Data Sent

The smoke client sent these requests without authentication cookies or authorization headers:

```text
GET /
GET /back-office
GET /api/posts/2026-07-26/feed?size=5
GET /api/posts/2026-07-26/feed?cursor=not-a-cursor&size=5
GET /api/accounts/2026-07-26/admin?page=0&size=25
GET /api/messages/2026-07-26/conversation/reader?size=25
GET /api/notifications/2026-07-26?size=25
GET /api/reports/2026-07-26?page=0&size=25
GET /api/admin/activity/2026-07-26?page=0&size=25
POST /api/reports/2025-09-03/example/resolve
Content-Type: application/json
{"resolution":"CLOSE_NO_ACTION","reason":"Verified policy review."}
DELETE /api/accounts/2025-09-03/example
DELETE /api/accounts/2026-07-26/example
POST /api/reports/2025-09-03
Content-Type: application/json
{"postId":"example","reason":"spam"}
POST /api/reports/2026-07-26
Content-Type: application/json
{"postId":"example","reason":"spam"}
```

The Back Office response body was also checked for `id="activityFilters"`,
`id="activityPrevious"`, and `/js/back-office.js`.

## Response Received

The running app returned HTTP status code: 200 for the public home, Back Office
shell, and stable feed. Protected endpoints returned the status codes shown below.

```text
GET /                                                        -> 200, 4035 bytes
GET /back-office                                             -> 200, 24540 bytes
GET /api/posts/2026-07-26/feed?size=5                        -> 200
GET /api/posts/2026-07-26/feed?cursor=not-a-cursor&size=5    -> 400
GET /api/accounts/2026-07-26/admin?page=0&size=25            -> 403
GET /api/messages/2026-07-26/conversation/reader?size=25     -> 403
GET /api/notifications/2026-07-26?size=25                    -> 403
GET /api/reports/2026-07-26?page=0&size=25                   -> 403
GET /api/admin/activity/2026-07-26?page=0&size=25            -> 403
POST /api/reports/2025-09-03/example/resolve                 -> 403
DELETE /api/accounts/2025-09-03/example                     -> 403
DELETE /api/accounts/2026-07-26/example                     -> 403
POST /api/reports/2025-09-03                               -> 403
POST /api/reports/2026-07-26                               -> 403
```

The empty stable feed response was:

```json
{
  "messages": null,
  "payload": {
    "items": [],
    "nextCursor": null
  },
  "requestId": null,
  "success": true
}
```

The Back Office shell checks all returned `True` for audit filters, audit paging, and the page
script mount.

## Pass / Fail

PASS. The final candidate app started against an isolated database, exposed the stable public feed
contract, preserved the legacy and additive protected API boundaries, and rendered the new audit
ledger controls. Reviewer-driven regressions proved privacy-safe deletion, durable audit and
notification retries, author-only edits, cancellation acknowledgement, and old-client response
compatibility. Production traffic was not interrupted.

## Evidence

- `./gradlew.bat :website:check` exited `0` on commit `349ba1c7`.
- Java test XML: 137 suites, 1,155 tests, 0 failures, 0 errors, 3 skipped.
- JavaScript tests: 231 passed, 0 failed.
- Runtime startup log identified local profile, port `8090`, isolated worktree, and PID `56928`.
- Focused review-remediation suites passed before the complete check, including account deletion,
  account/report moderation audit, admin activity, notifications, post editing/controllers,
  report controllers, uploads, and media playback.
- Runtime HTTP evidence is recorded verbatim in the response table above.
- After shutdown, only production PID `47288` remained listening on ports `8080`/`8090`.

## Bugs / Follow-ups

- The local profile's startup catch-up attempted an OpenStreetMap import and received an external
  HTTP `504`; the exception was logged and contained, and it did not affect startup or any tested
  route.
- Authenticated moderator mutations were not performed against the empty isolated database.
  Their actor/reason/before/after behavior, validation boundaries, audit-persistence `503`, and UI
  payload helpers are covered by the passing Java and JavaScript test suites.
