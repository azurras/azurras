# WFL and Location Imports Issues 1169-1175 Test Report

## Document Status

complete

## Story/Issue

GitHub issues `cbell504/website#1169` through `#1175`.

## Branch

- Repository: `A:\Projects\christopherbell.dev-worktrees\wfl-location-imports-1169-1175`
- Branch: `codex/wfl-location-imports-1169-1175`
- Commits under test: `d61ea463`, `b69f390b`, `54df92ae`, `02112be5`, `7a5b02ee`, and final head `c106eda9`
- Pull request: `cbell504/website#1256`

## App / Environment

- App: packaged `christopherbell.dev` Spring Boot application
- Profiles: `local,deploy-smoke`
- Candidate base URL: `http://127.0.0.1:8091`
- Production URL preserved during testing: `http://127.0.0.1:8080`
- Disposable MongoDB databases: `christopherbell_batch6_smoke` and `christopherbell_batch6_smoke_2`
- Mail delivery: disabled
- WFL scheduled import: disabled by `deploy-smoke`

## Local Run Details

The final packaged application was built with `\.\gradlew.bat :website:bootJar` and started hidden with the equivalent command:

```powershell
java -jar website\build\libs\website.jar `
  --spring.profiles.active=local,deploy-smoke `
  --server.port=8091 `
  --spring.mongodb.database=christopherbell_batch6_smoke_2 `
  --app.mail.enabled=false `
  --command-center.enabled=false
```

The first diagnostic process was PID `46368`. It exposed a global-security allowlist gap for the new public freshness route and was stopped. After the allowlist fix and rebuild, the final process was PID `53472`. It was stopped after testing. Both disposable databases were dropped successfully. Production remained available on port `8080` throughout.

## Test Cases

| Case | Behavior | Result |
| --- | --- | --- |
| 1 | Full Java, JavaScript, packaging, and sensor-runtime gate | PASS |
| 2 | WFL coordinate and top-rated data access use bounded/indexed queries | PASS |
| 3 | Manual and scheduled imports share one owner-scoped Mongo lease | PASS |
| 4 | Import preview token is short-lived, one-time, and operator-bound | PASS |
| 5 | Apply re-fetches and rejects a changed checksum before writes | PASS |
| 6 | Import status stores lifecycle, timing, counts, trigger, actor, and bounded error category | PASS |
| 7 | Duplicate preview identifies candidates and stable survivor | PASS |
| 8 | Duplicate apply validates every group version before any delete | PASS |
| 9 | Public freshness exposes only source, refresh time, current state, and 47 covered cities | PASS |
| 10 | Invalid WFL source, lease, metro, city ownership, or bounding-box configuration fails validation | PASS |
| 11 | Unchanged ZIP dataset checksum takes the no-write path and reports observability fields | PASS |
| 12 | Public home and WFL pages render from the candidate build | PASS |
| 13 | Protected import and dedupe operator APIs reject anonymous callers | PASS |
| 14 | Browser renders freshness on picks and top-rated pages without console errors | PASS |
| 15 | Candidate shutdown, disposable database cleanup, and production isolation | PASS |
| 16 | Async media controller test waits for streaming completion before dispatch assertions | PASS |

## Data Sent

The HTTP smoke sent no authentication cookie or authorization header:

```text
GET /
GET /wfl
GET /wfl/top-rated
GET /api/whatsforlunch/restaurant/2026-07-26/freshness
GET /api/whatsforlunch/restaurant/2026-07-26/import/openstreetmap/status
POST /api/whatsforlunch/restaurant/2026-07-26/import/openstreetmap/preview
GET /api/whatsforlunch/restaurant/2026-07-26/dedupe-names/preview
```

The browser loaded `/wfl` and `/wfl/top-rated` from the candidate app and inspected the accessible `Restaurant data freshness` landmark and browser console.

## Response Received

The running app returned the following HTTP results. The public freshness status code: 200. Its response body and the protected-boundary results are recorded below.

```text
GET /                                                                  -> 200
GET /wfl                                                              -> 200
GET /wfl/top-rated                                                    -> 200
GET /api/whatsforlunch/restaurant/2026-07-26/freshness                -> 200
GET /api/whatsforlunch/restaurant/2026-07-26/import/openstreetmap/status -> 403
POST /api/whatsforlunch/restaurant/2026-07-26/import/openstreetmap/preview -> 403
GET /api/whatsforlunch/restaurant/2026-07-26/dedupe-names/preview     -> 403
GET http://127.0.0.1:8080/                                            -> 200
```

The final freshness body was:

```json
{
  "payload": {
    "source": "OpenStreetMap",
    "lastRefreshedOn": null,
    "current": false,
    "currentWithinDays": 45,
    "cityCoverage": ["Addison, TX", "Austin, TX", "... 45 more cities"]
  },
  "success": true
}
```

The browser rendered `OpenStreetMap Not yet imported`, `47 covered cities`, and no console errors on both tested WFL page families.

## Pass / Fail

PASS. The final candidate enforces preview-before-apply imports and duplicate cleanup, prevents overlapping import writers with an owner-scoped lease, reports durable operator status without raw exception text, exposes a strictly public freshness DTO, validates WFL configuration, and makes unchanged ZIP imports a reported no-op. The candidate ran only on port `8091`; production remained on `8080` and was not restarted.

## Evidence

- Final `\.\gradlew.bat :website:check`: `BUILD SUCCESSFUL` in 1m 24s.
- Post-CI-fix `\.\gradlew.bat :website:check` at `c106eda9`: `BUILD SUCCESSFUL` in 1m 25s.
- `ProgressiveMediaControllerTest.readyDerivativeUsesTheStreamingResponseHandler` passed four forced focused executions after adding the missing async-result wait.
- Java XML: 1,170 tests, 0 failures, 0 errors, 3 skipped.
- JavaScript JUnit XML: 233 tests, 0 failures.
- Changed `back-office.js`, `whats-for-lunch.js`, `wfl-list.js`, and `wfl-freshness.js` passed `node --check`.
- `git diff --check` passed before commits.
- Final packaged runtime PID `53472` started on `8091` with isolated database `christopherbell_batch6_smoke_2`.
- Browser evidence confirmed the freshness landmark and 47-city summary on `/wfl` and `/wfl/top-rated`, with an empty error-console result.
- Independent diff review added an in-loop renewable lease guard; focused tests prove a long-running apply renews again before later writes and checks ownership before every candidate mutation.
- Both disposable MongoDB databases returned `{ ok: 1, dropped: ... }` during cleanup.
- Candidate PID `53472` was stopped; production root continued to return `200`.

## Bugs / Follow-ups

- Resolved during runtime testing: the first candidate returned `403` for the intended public freshness API because the global security matcher had not been updated. A focused matcher and anonymous MVC regression test were added; the rebuilt runtime returned `200`, and the browser rendered the freshness UI.
- Resolved during CI: Ubuntu exposed a pre-existing async MockMvc race in `ProgressiveMediaControllerTest` while the same test passed on macOS. The failure artifact showed Spring Security header mutation racing the mocked body write and raising `ConcurrentModificationException`; waiting for the async result before dispatch removed the race, passed four forced focused runs, and preserved the production implementation unchanged.
- The isolated database intentionally contained no successful OpenStreetMap import, so the final public UI displayed the honest `Not yet imported` state. Successful/failure lifecycle transitions, lease contention, checksum mismatch, and safe error categories are covered by focused automated tests.
- Authenticated destructive import and duplicate apply operations were not run against production or the empty disposable database. Their operator binding, checksum/version conflict behavior, all-before-any validation, and protected controller boundary are covered by the passing test suites.
