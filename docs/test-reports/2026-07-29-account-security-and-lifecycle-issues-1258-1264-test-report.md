# Account Security and Lifecycle Issues 1258-1264 Test Report

## Document Status

complete

## Story/Issue

GitHub issues `azurras/christopherbell.dev` #1258 through #1264.

## Branch

`codex/all-open-issues-20260729`, based on `origin/main` commit `8405cd77d0f1743fe33d70cc80b47e37048090a0`; implementation commits `fc294f7d`, `1fe4fcdd`, and `a098da97`.

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

After independent code review, the amended build was launched again as hidden PID `43212` against disposable database `cbell_issue_1258_1264_review_20260729`. The second run verified padded legacy-password rejection work, deterministic concurrent legacy upgrades, and immediate validity of both concurrently issued bearer tokens. Runtime log: `A:\GradleUserHomes\cbdev-issues-1258-1307\runtime-8093-review-20260729-081641.out.log`. PID 43212 was stopped, port 8093 was confirmed free, and the second disposable database was dropped.

After final review identified a whole-document login save race, commit `a098da97` replaced it with a conditional atomic Mongo update and minted tokens from the returned current account. The final build ran as hidden PID `51028` against `cbell_issue_1258_1264_atomic_20260729`; account creation, current-format login, and authenticated `/me` all passed. Runtime log: `A:\GradleUserHomes\cbdev-issues-1258-1307\runtime-8093-atomic-20260729.out.log`. PID 51028 was stopped, port 8093 was confirmed free, and the exact disposable database was dropped.

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
| Legacy timing parity | Legacy wrong-password work is padded to the current PBKDF2 work factor | Pass |
| Concurrent legacy upgrade | Two valid concurrent logins converge on one deterministic current hash and both tokens remain valid | Pass |
| Login/moderation race | Login updates only credential/login fields and mints from the atomically returned current role/status/permissions | Pass |
| Cleanup | Fixture, process, and disposable DB removed | Pass |

## Data Sent

1. `GET /` in a PowerShell web session to obtain `XSRF-TOKEN`.
2. `POST /api/accounts/2024-12-15/create` with JSON content, `X-XSRF-TOKEN`, and `{"firstName":"Batch","lastName":"Verifier","email":"batch1264b@example.test","password":"correct-horse-battery-staple","username":"batch1264b"}`.
3. `POST /api/accounts/2024-12-15/login` for an unknown email and for the test account with the same wrong password.
4. Correct login, then `GET /api/accounts/2025-09-03/me` with its bearer token.
5. Direct disposable-database status update from ACTIVE to SUSPENDED, then the same bearer request again.
6. Restore to ACTIVE, promote to ADMIN, login, then `DELETE /api/accounts/2026-07-26/{id}` with Authorization and no request Content-Type.
7. `POST /api/accounts/2024-12-15/login` with malformed body `{"email":`.
8. Three wrong-password logins against a legacy 65,536-iteration fixture and three logins for an unknown account.
9. Two concurrent valid logins against the same legacy credential, followed by `GET /api/accounts/2025-09-03/me` with each returned bearer token.

## Response Received

- HTTP status code: `201 Created`; `Location: /api/accounts/2025-09-03/f36d2681-047d-4e9f-a41a-70a2bb159008`; response body had `success=true` and no `isApproved` or `approvedBy`.
- Stored credential: current prefix `pbkdf2-sha256$210000$`; no `passwordSalt` field.
- Both failed logins returned status code 401 and the identical response body `{"messages":[{"code":"INVALID_TOKEN","description":"Login failed."}],"payload":null,"requestId":null,"success":false}`.
- Correct login and `/me` before mutation returned status code 200; the same bearer after suspension returned status code 401.
- Bodyless ADMIN DELETE returned status code 200 with deletion status COMPLETE and six completed steps.
- Malformed JSON returned status code 400 with response body description `The request body is malformed or invalid.`
- Three legacy wrong-password requests took 176.4 ms total versus 145.3 ms for three unknown-account requests, a 1.21 ratio after padding both paths to the current 210,000-iteration work factor.
- Concurrent legacy logins both returned 200; both bearer tokens immediately returned 200 from `/me`; the stored credential used the current self-describing format and no legacy salt remained.
- Final atomic-path smoke returned root 200, create 201 with `Location`, login 200, and authenticated `/me` 200. The stored account had the current 210,000-iteration format, no legacy salt, and a populated `lastLoginOn`.
- Post-delete database query returned zero matching test accounts.

## Pass / Fail

All runtime cases passed. No listener, account fixture, or disposable database remains.

Supporting checks passed after all review amendments: `:cbell-lib:test` 101 tests with 0 failures and 0 errors; final `:website:check` 1,392 Java tests with 0 failures, 0 errors, and 3 skipped; `:website:jsTest` 269 tests with 0 failures and 0 errors; `:website:verifySensorRuntime`; and `node --check` for `back-office.js` and `music.js`. The final combined check reported `BUILD SUCCESSFUL` in 1m42s.

## Evidence

- Runtime log named under Local Run Details.
- Post-review runtime log named under Local Run Details.
- Java XML: `website/build/test-results/test/` and `cbell-lib/build/test-results/test/`.
- JS XML: `website/build/test-results/jsTest/results.xml`.
- Cleanup output: `STOPPED_PID=35468`, `STOPPED_PID=43212`, PID 51028 stopped, `PORT_8093_LISTENING=False`, all database drops `{ ok: 1 }`, and all disposable databases absent.
- Production continuity: `ChristopherBellDev` remained Running; local port-8080 root and `https://www.christopherbell.dev/` returned status code 200.

## Bugs / Follow-ups

The first alternate launch exposed that `application-local.yml` has an explicit `spring.mongodb.database`, so a database name embedded only in the URI is insufficient. It was stopped immediately. Its one test account was deleted, all 20 pre-existing accounts were restored to their prior effective approval state, and the premature V008 record was removed. Verification showed 20 accounts, 20 with `isApproved=true`, zero test accounts, and zero V008 records. All accepted reruns set both Mongo variables and proved V008 existed only in their disposable databases. Independent review identified and corrected legacy-versus-unknown rejection work disparity, current-format authentication with a null legacy salt, nondeterministic concurrent legacy upgrades, and an unconstrained login save that could overwrite simultaneous moderation. No unresolved Batch 1 defect remains.
