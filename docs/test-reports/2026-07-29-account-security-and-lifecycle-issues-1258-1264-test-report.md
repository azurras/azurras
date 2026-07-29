# Account Security and Lifecycle Issues 1258-1264 Test Report

## Document Status

complete

## Story/Issue

GitHub issues `azurras/christopherbell.dev` #1258 through #1264.

## Branch

`codex/all-open-issues-20260729`, based on `origin/main` commit `8405cd77d0f1743fe33d70cc80b47e37048090a0`; Batch 1 working tree before its implementation commit.

## App / Environment

- Spring Boot website on native Windows production/development host.
- Profile `local`; alternate base URL `http://127.0.0.1:8093`.
- Disposable Mongo database `cbell_issue_1258_1264_20260729`.
- Explicit `SPRING_PROFILES_ACTIVE=local`, `SERVER_PORT=8093`, `SPRING_MONGODB_URI=mongodb://127.0.0.1:27017/cbell_issue_1258_1264_20260729`, `SPRING_MONGODB_DATABASE=cbell_issue_1258_1264_20260729`, `APP_MAIL_ENABLED=false`, and a local-only strong JWT secret.
- Production port 8080 and `ChristopherBellDev` were never stopped.

## Local Run Details

Built with:

```powershell
$env:GRADLE_USER_HOME='A:\GradleUserHomes\cbdev-issues-1258-1307'
.\gradlew.bat :website:bootJar --console=plain
```

Launched as hidden PID `35468` with `java -jar website\build\libs\website.jar` and the explicit environment above. `/` and `/actuator/health/readiness` returned 200; readiness body was `{"status":"UP"}`. Runtime log: `A:\GradleUserHomes\cbdev-issues-1258-1307\runtime-8093-20260729-075704.out.log`.

PID 35468 was stopped afterward, port 8093 was confirmed free, and the disposable database was dropped and confirmed absent.

## Test Cases

| Case | Expected behavior | Result |
|---|---|---|
| Startup/migration | Ready app; V008 only in disposable DB | Pass |
| Create | 201, Location, no approval fields | Pass |
| Password storage | Current PBKDF2 format; no legacy salt | Pass |
| Failed login | Unknown and wrong password are indistinguishable | Pass |
| Revocation | Persisted status mutation immediately rejects bearer | Pass |
| Bodyless DELETE | ADMIN delete without Content-Type reaches controller | Pass |
| Malformed JSON | Stable public parser error | Pass |
| Cleanup | Fixture, process, and disposable DB removed | Pass |

## Data Sent

1. `GET /` in a PowerShell web session to obtain `XSRF-TOKEN`.
2. `POST /api/accounts/2024-12-15/create` with JSON content, `X-XSRF-TOKEN`, and `{"firstName":"Batch","lastName":"Verifier","email":"batch1264b@example.test","password":"correct-horse-battery-staple","username":"batch1264b"}`.
3. `POST /api/accounts/2024-12-15/login` for an unknown email and for the test account with the same wrong password.
4. Correct login, then `GET /api/accounts/2025-09-03/me` with its bearer token.
5. Direct disposable-database status update from ACTIVE to SUSPENDED, then the same bearer request again.
6. Restore to ACTIVE, promote to ADMIN, login, then `DELETE /api/accounts/2026-07-26/{id}` with Authorization and no request Content-Type.
7. `POST /api/accounts/2024-12-15/login` with malformed body `{"email":`.

## Response Received

- HTTP status code: `201 Created`; `Location: /api/accounts/2025-09-03/f36d2681-047d-4e9f-a41a-70a2bb159008`; response body had `success=true` and no `isApproved` or `approvedBy`.
- Stored credential: current prefix `pbkdf2-sha256$210000$`; no `passwordSalt` field.
- Both failed logins returned status code 401 and the identical response body `{"messages":[{"code":"INVALID_TOKEN","description":"Login failed."}],"payload":null,"requestId":null,"success":false}`.
- Correct login and `/me` before mutation returned status code 200; the same bearer after suspension returned status code 401.
- Bodyless ADMIN DELETE returned status code 200 with deletion status COMPLETE and six completed steps.
- Malformed JSON returned status code 400 with response body description `The request body is malformed or invalid.`
- Post-delete database query returned zero matching test accounts.

## Pass / Fail

All runtime cases passed. No listener, account fixture, or disposable database remains.

Supporting checks passed: `:cbell-lib:test` 100 tests; final `:website:check` 1,389 Java tests with 0 failures, 0 errors, 3 skipped; `:website:jsTest` 269 tests; `:website:verifySensorRuntime`; and `node --check` for `back-office.js` and `music.js`. Final check reported `BUILD SUCCESSFUL` in 1m31s.

## Evidence

- Runtime log named under Local Run Details.
- Java XML: `website/build/test-results/test/` and `cbell-lib/build/test-results/test/`.
- JS XML: `website/build/test-results/jsTest/results.xml`.
- Cleanup output: `STOPPED_PID=35468`, `PORT_8093_LISTENING=False`, database drop `{ ok: 1 }`, `DISPOSABLE_DB_EXISTS=false`.
- Production continuity: `ChristopherBellDev` remained Running; local port-8080 root and `https://www.christopherbell.dev/` returned status code 200.

## Bugs / Follow-ups

The first alternate launch exposed that `application-local.yml` has an explicit `spring.mongodb.database`, so a database name embedded only in the URI is insufficient. It was stopped immediately. Its one test account was deleted, all 20 pre-existing accounts were restored to their prior effective approval state, and the premature V008 record was removed. Verification showed 20 accounts, 20 with `isApproved=true`, zero test accounts, and zero V008 records. The accepted rerun set both Mongo variables and proved V008 existed only in the disposable database. No unresolved Batch 1 defect remains.
