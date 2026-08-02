# christopherbell.dev Shared Library Boundaries Test Report

## Document Status

complete

## Story/Issue

Approved repository-wide website performance, scalability, and shared-library optimization campaign; shared library boundaries phase.

## Branch

- Spoke branch: `codex/shared-library-boundaries`
- Commit under test: `52c5b4e0c1d8bd6683ffcde00fb3975a20a74b4c`
- Base commit: `95d805658beaa4c62a8b5e56af9bbf1c0aca66a6`
- Pull request: `azurras/christopherbell.dev#1338`
- Merged commit: `2b40bd860d9e4e05aa18b4dd63e13a390d41208e`

## App / Environment

- App: `christopherbell.dev` Spring Boot website.
- Candidate URL: `http://127.0.0.1:8094`.
- Profiles: `local,deploy-smoke`.
- Disposable Mongo database: `christopherbell_shared_lib_verify_20260801_3789f765`.
- Scheduling: disabled by the `deploy-smoke` profile.
- Mail and music: disabled for the candidate.
- Production listener: port 8080, PID `33336` throughout candidate testing;
  automatically rotated to PID `33024` after merge.
- Services: MongoDB, ChristopherBellDev, and cloudflared were Running/Automatic before the candidate test.

## Local Run Details

The candidate JAR ran as PID `30628` from
`A:\Projects\christopherbell.dev-worktrees\shared-library-boundaries`.
The sanitized start form was:

```powershell
java -jar website\build\libs\website.jar `
  --spring.profiles.active=local,deploy-smoke `
  --server.port=8094 `
  --spring.mongodb.database=christopherbell_shared_lib_verify_20260801_3789f765 `
  --spring.data.mongodb.database=christopherbell_shared_lib_verify_20260801_3789f765 `
  --app.mail.enabled=false `
  --app.music.enabled=false
```

The application started in 7.116 seconds and Spring discovered 38 MongoDB
repository interfaces. After testing, PID 30628 was stopped, port 8094 was
confirmed free, the exact disposable database was dropped successfully, and
temporary logs were deleted. Production port 8080 remained PID 33336.

## Test Cases

| Case | Expected | Result |
| --- | --- | --- |
| Application context | Moved cursor, lease, and WFL workflow components are discovered without duplicate or missing beans | Pass |
| Public pages | Home, signup, and login pages render | Pass: status code 200 |
| Cursor pagination | Public feed accepts `limit=1` and returns a bounded page | Pass: status code 200, 90-byte response |
| WFL runtime | Public freshness route resolves through the WFL feature after workflow ownership changes | Pass: status code 200, 890-byte response |
| Account creation | Browser-style CSRF-protected account creation succeeds in the disposable database | Pass: status code 201 |
| JWT login | Direct website-scoped JJWT API/runtime creates a usable login response | Pass: status code 200 with non-empty token payload |
| Lease behavior | Acquisition, contention, exact-expiry takeover, renewal, ownership loss, release, and collector lifecycle remain correct | Pass: 12 focused library lease tests plus website collector suites |
| Test fixture isolation | Website controller tests resolve `TestUtil`, main production JAR does not contain it, and test-fixture JAR does | Pass |
| Dependency ownership | `cbell-lib` runtime has no JJWT; website runtime resolves API, implementation, and Jackson adapter at 0.13.0 | Pass |
| Retired packages | Old pagination, lease, and workflow package names are absent | Pass |
| Full candidate gate | Both module checks rerun without cache-only evidence | Pass |
| Cleanup and production isolation | Candidate, temporary logs, and disposable data are removed; production listener is unchanged | Pass |
| Pull request and main CI | Linux, macOS, Windows, dependency review, CodeQL, and post-merge checks pass | Pass |
| Production delivery | New listener serves local and public health, cursor-feed, WFL, and public-site routes | Pass |

## Data Sent

Read-only HTTP GET requests were sent to:

- `/`
- `/login`
- `/signup`
- `/api/posts/2026-07-26/feed?limit=1`
- `/api/whatsforlunch/restaurant/2026-07-26/freshness`

A disposable account-create JSON request was sent to
`POST /api/accounts/2024-12-15/create` with unique test-only name, email,
username, password, and federation disabled. The browser-style request included
the `XSRF-TOKEN` cookie and matching `X-XSRF-TOKEN` header obtained from the
signup page. A matching login JSON request was sent to
`POST /api/accounts/2024-12-15/login` with `Accept: application/json`.
No production credentials or production data were used.

## Response Received

- Status code: 200 OK for `/`, `/login`, `/signup`, the cursor feed, and WFL freshness.
- The cursor feed returned a 90-byte empty bounded response.
- WFL freshness returned an 890-byte response.
- Status code: 201 Created for the account-create request with a 463-byte account response.
- Status code: 200 OK for login with a non-empty JWT payload.
- Log excerpt: Tomcat started on port 8094, 38 Mongo repositories were discovered, and startup completed in 7.116 seconds.
- Candidate and production listeners were simultaneously visible as PID 30628 on port 8094 and PID 33336 on port 8080.
- Cleanup returned MongoDB `{ "ok": 1, "dropped": "christopherbell_shared_lib_verify_20260801_3789f765" }` and left port 8094 free.
- After merge, production rotated from PID 33336 to PID 33024. Liveness
  returned status code 200 throughout observation; readiness briefly returned
  503 during the restart window and then recovered to status code 200 with
  `{"status":"UP"}`.
- Local production home, login, liveness, readiness, cursor feed, WFL freshness,
  `robots.txt`, and `sitemap.xml` returned status code 200 on PID 33024.
- Public home, login, cursor feed, WFL freshness, `robots.txt`, and `sitemap.xml`
  returned status code 200.

## Pass / Fail

Pass. Stable pagination and generic Mongo lease infrastructure now compile and
test from `cbell-lib`; the workflow engine is WFL-owned; `TestUtil` is test-only;
and JJWT is website-owned. The full build, runtime flows, dependency evidence,
cleanup, and production isolation all passed.

The fresh gate ran `:cbell-lib:check :website:check --rerun-tasks` successfully
in 3 minutes 28 seconds across 24 executed tasks. It included 121 library Java
tests and 1,609 website Java tests, for 1,730 Java tests total with 0 failures,
0 errors, and 3 intentional skips, plus 311 JavaScript tests with 0 failures.

## Evidence

- Full gate: `BUILD SUCCESSFUL in 3m 28s`, 24 tasks executed.
- Java tests: 1,730 passed across both modules; 0 failures; 0 errors; 3 intentional skips.
- JavaScript tests: 311 passed; 0 failures.
- Focused Mongo lease suite: 12 passed, including two-coordinator exact-expiry arbitration.
- Focused WFL workflow suite: 11 executor tests and 2 WFL integration tests passed.
- JJWT tests: 27 permission/filter tests passed.
- Controller fixture verification: all `*ControllerTest` cases passed.
- Production JAR inspection: no `dev/christopherbell/libs/test/TestUtil.class`.
- Test-fixture JAR inspection: `dev/christopherbell/libs/test/TestUtil.class` present.
- `cbell-lib` runtime dependency report: no `io.jsonwebtoken` artifacts.
- Website runtime dependency insight: `jjwt-api`, `jjwt-impl`, and `jjwt-jackson` all resolve to 0.13.0.
- Zero-reference searches found no retired pagination, lease, or workflow package names.
- Independent read-only review approved HEAD `52c5b4e0` with no findings.
- Candidate startup: PID 30628, port 8094, 7.116 seconds.
- Production isolation: port 8080 remained PID 33336.
- Cleanup: candidate stopped, database drop returned `ok: 1`, runtime artifacts removed.
- PR #1338 required checks: Linux, macOS, Windows, dependency review, all
  CodeQL analyses, and the aggregate CodeQL gate passed at head `52c5b4e0`.
- PR #1338 merged as `2b40bd860d9e4e05aa18b4dd63e13a390d41208e`.
- Post-merge CI Build `30730666489` and CodeQL `30730666478` passed.
- Production listener rotated from PID 33336 to PID 33024; MongoDB,
  ChristopherBellDev, and cloudflared remained Running/Automatic.

## Bugs / Follow-ups

No defect was found. The live manual collector endpoint was not invoked because
it performs bounded but real third-party network collection; the same manual
lease path is covered by focused service/controller tests, and the moved lease
components were proven in the running application context.
