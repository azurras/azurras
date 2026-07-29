# christopherbell.dev Authentication Request Efficiency Test Report

## Document Status

blocked

The corrected isolated candidate passed every functional, query-count, and cleanup
criterion. Closure is blocked only on separate authority to inspect and remove one
known disposable test identity that an earlier misconfigured run may have created in
the production-named database.

## Story/Issue

[christopherbell.dev Performance, Scalability, and Library Optimization](../work/2026-07-29-christopherbell-dev-performance-scalability-library-optimization.md),
authentication request-efficiency plan.

## Branch

- Candidate branch: `codex/performance-authentication-efficiency`
- Candidate commit tested at runtime: `ed87db4087ac0776d9b9e0eb11a14f4b526dfe85`
- Corrected full-gate commit: the same `ed87db40` candidate; `:website:check`
  passed after the proxy-runtime fix.
- Baseline commit: `e3afbf3c9eeb65525f573f299f82287ef8665554`

## App / Environment

- App: `christopherbell.dev` Spring Boot website.
- Profile: `local` with mail disabled and a throwaway JWT secret.
- Baseline URL: `http://127.0.0.1:8091`.
- Candidate URL: `http://127.0.0.1:8092`.
- Corrected baseline database:
  `christopherbell_perf_auth_before_safe_20260729`.
- Corrected candidate database:
  `christopherbell_perf_auth_after_safe_20260729`.
- Both the Mongo URI and the higher-precedence `spring.mongodb.database`
  property named the exact disposable database.
- Port 8080 was never targeted by a test command. Its externally managed listener
  rotated from PID `60136` to PID `48420` while testing was underway.

## Local Run Details

The baseline JAR ran as PID `48412` on 8091 and the candidate JAR ran as PID
`50412` on 8092. Startup logs showed Tomcat initialized and started on the assigned
alternate port; candidate PID 50412 reported `Started Application` after 5.729
seconds. Each exact disposable database was verified empty before signup and
populated only by its matching run. Both PIDs were stopped explicitly, both ports
were confirmed free, and only the two exact corrected disposable databases were
dropped and verified empty.

The same temporary, property-gated Mongo command listener and request-correlation
filter was applied to both worktrees. It recorded request ID, path, collection, and
command only. Secret scans found no password, JWT secret, bearer value, or browser
authentication-cookie value. The diagnostic source was removed with `apply_patch`
and never committed.

## Test Cases

| Case | Expected | Result |
| --- | --- | --- |
| Full candidate gate | Java, JavaScript, packaging, sensor runtime, and policy pass on the post-proxy-fix commit | Pass: `BUILD SUCCESSFUL in 1m 38s` |
| Static assets with credentials | Real unversioned and SHA-versioned assets return 200 with zero auth Mongo commands | Pass |
| Normal cookie authentication | One `browser_sessions` read and zero `accounts` reads | Pass |
| Activity coalescing | Five requests inside five minutes perform no session write | Pass |
| Baseline comparison | Baseline shows the removed account read and full session update | Pass |
| Login/logout | Browser cookies set at login and zero-aged at logout | Pass |
| Revoked cookie | Old cookie fails closed after logout | Pass: 401 and authentication cookies cleared |
| Descriptive latency | Same endpoint, headers, host, and 30-request method on both builds | Pass; no statistical claim made |
| Corrected data cleanup | Stop exact PIDs and drop only the two exact safe databases | Pass |
| Discarded initial run cleanup | Assess/remove possible production-named test identity | Blocked pending authority |

## Data Sent

- `GET /login` to obtain the CSRF cookie.
- `POST /api/accounts/2024-12-15/create` with a fresh disposable USER account.
- Browser-cookie login at `POST /api/accounts/2024-12-15/login`.
- `GET /js/app.js` and the real build-versioned
  `GET /<commit-sha>/js/app.js` with a stale cookie and bearer header.
- Cookie-authenticated `GET /api/accounts/2024-12-15`. A USER is correctly denied
  by the ADMIN-only endpoint after authentication, isolating authentication reads
  from controller-level account reads.
- Five repeated copies of that protected request within five minutes.
- Thirty sequential copies of the protected request for descriptive latency.
- `POST /api/accounts/2024-12-15/logout`, then the protected request with the old
  revoked cookie.

Credentials, cookie values, bearer values, password text, and the JWT secret were
not retained in this report.

## Response Received

| Flow | Baseline | Candidate |
| --- | --- | --- |
| Anonymous login page | 200 | 200 |
| Browser login | 200; HttpOnly auth cookie and UI-state cookie set | 200; same transition |
| Unversioned static asset | 200; auth commands 0 | 200; auth commands 0 |
| Real SHA-versioned static asset | 200; auth commands 0 | 200; auth commands 0 |
| Protected account authorization | 403; session find 1, account find 1 | 403; session find 1, account find 0 |
| Five interactive requests | each 403; session find 1, account find 1, session update 1 | each 403; session find 1, account find 0, session writes 0 |
| Logout | 200; auth cookies zero-aged | 200; auth cookies zero-aged |
| Old cookie after logout | 401; auth cookies cleared | 401; auth cookies cleared |

The actual asset paths were
`/e3afbf3c9eeb65525f573f299f82287ef8665554/js/app.js` and
`/ed87db4087ac0776d9b9e0eb11a14f4b526dfe85/js/app.js`.

## Pass / Fail

All corrected isolated functional and performance cases passed. The candidate
removed one account read from ordinary cookie authentication and removed repeated
full session updates inside the five-minute activity window. The candidate's local
p50 was effectively unchanged; its observed p95 was lower. These are descriptive
local samples only.

Overall report status remains blocked because the discarded URI-only baseline run
may have created one known test account in the production-named database. No
production database inspection or cleanup occurred after discovery.

## Evidence

### Automated verification

```text
GRADLE_USER_HOME=...performance-authentication-20260729-task5
./gradlew.bat :website:test --tests dev.christopherbell.configuration.JwtAuthenticationFilterTest --tests dev.christopherbell.configuration.security.browser.* --tests dev.christopherbell.account.* --tests dev.christopherbell.report.moderation.*
PASS in 26.6 s

./gradlew.bat :website:check --offline
PASS in 1m 38s on ed87db40 after the proxy-runtime fix
```

An earlier final-gate attempt paired `--offline` with a new empty Gradle cache and
failed before compilation because the Spring Boot plugin was unavailable locally.
The warmed isolated cache rerun above is the valid result.

### Sanitized startup excerpt

```text
PID 50412: Tomcat initialized with port 8092
PID 50412: Tomcat started on port 8092 with context path '/'
PID 50412: Started Application in 5.729 seconds
```

### Sanitized correlated command excerpts

```text
baseline-safe-protected  /api/accounts/2024-12-15  browser_sessions find
baseline-safe-protected  /api/accounts/2024-12-15  accounts         find

candidate-safe-protected /api/accounts/2024-12-15  browser_sessions find
```

For baseline interactive requests 1 through 5, each correlation contained exactly:

```text
browser_sessions find
accounts         find
browser_sessions update
```

For candidate interactive requests 1 through 5, each correlation contained exactly
one `browser_sessions find` and no `accounts` event or session write. Candidate
logout contained `browser_sessions find` followed by `browser_sessions delete`; the
old-cookie request then contained one session find and returned 401.

No Mongo event exists for either candidate static-asset correlation. The retained
HTTP summary recorded status 200 for both actual paths. Raw HTTP headers and
individual latency samples were not written to disk; their sanitized status/cookie
transitions and aggregate values are preserved here and must not be represented as
raw captures.

### Descriptive latency

| Build | n | p50 ms | p95 ms | min ms | max ms |
| --- | ---: | ---: | ---: | ---: | ---: |
| Baseline | 30 | 4.914 | 8.735 | 3.325 | 19.313 |
| Candidate | 30 | 4.927 | 5.606 | 3.846 | 14.832 |

The observed p95 difference was 3.129 ms; the p50 difference was negligible.

### Cleanup evidence

`mongosh` verified and dropped only:

```text
christopherbell_perf_auth_before_safe_20260729  ok: 1
christopherbell_perf_auth_after_safe_20260729   ok: 1
```

Reconnects to both exact names returned zero collections. Candidate and baseline
tracked worktrees were clean after diagnostic removal; 8091 and 8092 had no listener.

## Bugs / Follow-ups

1. Candidate startup initially exposed a real Spring proxy defect: a final
   `MongoBrowserSessionActivityStore` could not be subclassed for repository
   exception translation. Commit `ed87db40` made only that adapter proxyable and
   added a real Spring class-proxy context regression. Nineteen focused tests and
   the final full website gate passed afterward.
2. The first baseline/candidate attempts supplied only the database component of
   the Mongo URI. `application-local.yml` supplied a higher-precedence explicit
   database, so those runs were invalid and discarded. Corrected runs supplied both
   properties and proved isolation before mutation.
3. Separate authority is required before inspecting or deleting the known initial
   test identity:
   - account ID: `87bb9e7d-ed0b-4d97-acbb-8cc15ab7e77b`
   - email: `perf-auth-20260729@example.test`
   - username: `perfauthbaseline20260729`
   - created: `2026-07-29T17:21:02.421Z`
   - creation endpoint: `POST /api/accounts/2024-12-15/create`
   - a later candidate attempt with the same email returned `RESOURCE_EXISTS` and
     did not create a second account.

Do not use those identifiers against production without explicit user authority.
