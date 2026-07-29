# Issue 1261 True-Error Logging Test Report

## Document Status

complete

## Story/Issue

`azurras/christopherbell.dev` issue #1261: return stable request errors and reserve error-level stack traces for genuine server or infrastructure failures.

## Branch

Tested `codex/all-open-issues-20260729` at `ad722de4a806fdb2f208ff4b93eea23e62c3e638` in the isolated worktree `A:\Projects\christopherbell.dev-worktrees\all-open-issues-20260729`.

## App / Environment

- Application: `christopherbell.dev` Spring Boot website.
- Profile: `prod`.
- Alternate port/base URL: `18081`, `http://localhost:18081`.
- Production port: `8080`; remained live and was not stopped, restarted, reconfigured, or rebound.
- Database: explicit loopback MongoDB URI against the local host database.
- Isolation: task-owned temporary JWT/federation values; mail, inaccessible music/shared-folder/command-center host integrations, application scheduling, and WFL monthly import disabled only for the alternate-port process.
- Build isolation: `%TEMP%\christopherbell-dev-gradle-issue-1261` as `GRADLE_USER_HOME`.

## Local Run Details

Automated validation ran first:

```powershell
$env:GRADLE_USER_HOME = Join-Path $env:TEMP 'christopherbell-dev-gradle-issue-1261'
.\gradlew.bat :cbell-lib:test --tests 'dev.christopherbell.libs.api.controller.ControllerExceptionHandlerTest' :website:test --tests 'dev.christopherbell.configuration.security.AsyncDispatcherSecurityIntegrationTest' --tests 'dev.christopherbell.sharedfolder.media.ProgressiveMediaControllerTest'
.\gradlew.bat --no-daemon :website:check
```

The alternate-port process used this explicit invocation shape; task-owned secret values are redacted and were not copied from production:

```powershell
$env:SPRING_PROFILES_ACTIVE = 'prod'
$env:SERVER_PORT = '18081'
$env:SPRING_DATA_MONGODB_URI = 'mongodb://127.0.0.1/<local-database>'
$env:APP_MUSIC_ENABLED = 'false'
$env:APP_SHARED_FOLDER_ENABLED = 'false'
$env:COMMAND_CENTER_ENABLED = 'false'
$env:APP_SCHEDULING_ENABLED = 'false'
$env:WFL_RESTAURANT_IMPORT_MONTHLY_ENABLED = 'false'
# Mail disabled; JWT and federation settings used task-owned temporary values.
.\gradlew.bat :website:bootRun
```

The final observation process was Java PID 40004. It ran for 38 seconds after startup, was stopped only after ownership of port 18081 was confirmed, and port 18081 was verified closed. Production port 8080 remained live on PID 51060 at the end of testing.

Runtime logs:

- `%TEMP%\christopherbell-dev-issue-1261-runtime\bootrun-retest-ad722de4.stdout.log`
- `%TEMP%\christopherbell-dev-issue-1261-runtime\bootrun-retest-ad722de4.stderr.log`
- `%TEMP%\christopherbell-dev-issue-1261-runtime\bootrun-retest-ad722de4-observation.stdout.log`

## Test Cases

| Case | Expected result | Result |
| --- | --- | --- |
| Protected initial request | Authentication remains required | PASS via production-chain integration test |
| Async/error redispatch | Redispatch continues without second denial | PASS via production-chain integration test |
| Authenticated async streaming | Response completes through async redispatch | PASS via production-chain integration test |
| Unacceptable response media | Stable JSON 406 with no resolver throwable warning | PASS at runtime |
| Malformed UUID path | Stable JSON 400 with no error stack | PASS at runtime |
| Valid absent UUID | Stable JSON 404 with no error stack | PASS at runtime |
| Malformed JSON | Stable JSON 400 with no error stack | PASS at runtime |
| Unsupported request media | Stable JSON 415 with no error stack | PASS at runtime |
| Home/security headers | 200 plus no-store and browser security headers | PASS at runtime |
| Old production error signature | No async denial, committed-response, `/error` cascade, or unexpected ERROR recurrence over more than two former recurrence intervals | PASS over 38 seconds |
| Genuine 500/503 classification | Causal ERROR and throwable retained | PASS via captured-log tests |

## Data Sent

- `POST http://localhost:18081/api/accounts/2024-12-15/login` with a JSON login body and `Accept: application/xml`.
- `GET http://localhost:18081/api/blog/v1/posts/not-a-uuid`.
- `GET http://localhost:18081/api/blog/v1/posts/6b8423dd-32d5-48fb-8172-b348a3484c9f` using a syntactically valid absent UUID.
- `POST http://localhost:18081/api/accounts/2024-12-15/login` with truncated JSON.
- The same login endpoint with `Content-Type: text/plain`.
- `GET http://localhost:18081/`.
- No credentials, tokens, production private keys, or sensitive request data were written to this report.

## Response Received

- Unacceptable `Accept`: status code 406, `application/json`, response body code `REQUEST_ERROR`, description `The requested response format is not available.`
- Malformed UUID: status code 400, `application/json`, response body code `REQUEST_ERROR`, description `The request is invalid.`
- Valid absent UUID: status code 404, `application/json`, response body code `RESOURCE_NOT_FOUND`, description `The requested resource was not found.`
- Malformed JSON: status code 400, `application/json`, response body code `REQUEST_ERROR`, description `The request body is malformed or invalid.`
- Unsupported media: status code 415, `application/json`, response body code `REQUEST_ERROR`, description `The request media type is not supported.`
- Home: status code 200, HTML response body, `Cache-Control: no-store, must-revalidate, no-cache, max-age=0`, `X-Content-Type-Options: nosniff`, `X-Frame-Options: SAMEORIGIN`, and Content Security Policy present.
- Each expected 4xx request had zero `ERROR` lines and zero throwable-bearing `WARN` lines in its isolated post-request log delta.
- The final 38-second log had zero `AuthorizationDeniedException`, committed-response, `/error`-cascade, or `ERROR` signatures.

## Pass / Fail

- PASS: focused combined command exited 0.
- PASS: serialized `:website:check` exited 0 in 1m54s. Parsed results recorded 1,400 website tests with 0 failures, 0 errors, and 3 skipped; the focused library suite recorded 7 tests with 0 failures/errors.
- PASS: `AsyncDispatcherSecurityIntegrationTest` recorded 3 tests with 0 failures/errors, covering protected initial request denial, async/error continuation, and authenticated streaming completion.
- PASS: all alternate-port HTTP and bounded-log acceptance cases listed above.
- PASS: alternate port closed after testing and production port remained live.

## Evidence

- Reviewed implementation commits: `825adad2`, `e52a3847`, `c5a4e894`, and `ad722de4`.
- Test XML: `website/build/test-results/test` contained 205 suites and 1,400 tests; `cbell-lib/build/test-results/test` contained 7 tests. Both had zero failures/errors.
- Final runtime log: `%TEMP%\christopherbell-dev-issue-1261-runtime\bootrun-retest-ad722de4-observation.stdout.log`.
- Verification ledger/report: `.superpowers/sdd/2026-07-29-issue-1261-true-error-logging/task-3-report.md` in the isolated spoke worktree.
- Final Git boundary: only the pre-existing modified `gradlew.bat` and untracked Gradle cache directories remained outside committed issue files; Task 3 created no source diff or commit.

## Bugs / Follow-ups

- No safe local authenticated credential/session was available for an interactive alternate-port streaming or Mission Control check. The production `SecurityConfig` integration tests provide the authenticated streaming, initial denial, and redispatch evidence without exposing a secret or adding a backdoor.
- No safe controlled live 500 endpoint existed. No backdoor was added. `ControllerExceptionHandlerTest` proves generic 500, framework 500, and known 503 remain causal `ERROR` records with throwables; direct runtime 500 coverage remains an explicit gap.
- Production deployment and live post-deploy verification remain pending publication, CI, merge, and the production-safe rollout step.
