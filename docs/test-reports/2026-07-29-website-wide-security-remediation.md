# Website-Wide Security Remediation Test Report

## Document Status

complete

## Story/Issue

Repository-wide security audit and remediation tracked by `docs/work/2026-07-28-website-wide-security-audit-and-remediation.md`.

## Branch

- Repository: `azurras/christopherbell.dev`
- Branch: `codex/security-audit-20260728`
- Commit under test: `5a2186ea5ea2b946faecead2b514f408bab6031e`
- Comparison base: `e3afbf3c9eeb65525f573f299f82287ef8665554`

## App / Environment

- App: `christopherbell.dev` Spring Boot application
- Host: native Windows development/production host
- Profiles: `prod,deploy-smoke`
- Candidate URL: `http://127.0.0.1:8081`
- Live production listener: port `8080`, left unchanged throughout candidate testing
- Database: local MongoDB at `127.0.0.1:27017`, isolated temporary database `christopherbell_security_candidate_5a2186ea`
- External side effects disabled: mail, federation discovery/inbound/outbound, sensors, power, and music integrations
- Candidate identity: `GIT_COMMIT=5a2186ea5ea2b946faecead2b514f408bab6031e`
- Authentication material: a long, ephemeral test-only JWT secret; no production secrets were used

## Local Run Details

The exact built artifact `website/build/libs/website.jar` was started with Java on the unused candidate port after setting the isolated database, ephemeral JWT secret, disabled integrations, profiles, and commit identity described above:

```powershell
java -jar website/build/libs/website.jar --server.port=8081 --spring.profiles.active=prod,deploy-smoke
```

The candidate process was PID `61400`. Its command line was inspected to confirm the worktree artifact and port `8081`. Startup logs showed Java 25.0.3, both requested profiles, a successful MongoDB connection, Tomcat listening on port 8081, and application startup in 5.583 seconds. After the checks, PID 61400 was revalidated by command line and stopped. Port 8081 was confirmed free; the live port 8080 remained owned by PID 60136.

The temporary database name was checked before deletion, dropped with MongoDB returning `{"ok":1,"dropped":"christopherbell_security_candidate_5a2186ea"}`, and confirmed absent afterward. It contained only test data and is not recoverable.

## Test Cases

| Case | Behavior exercised | Result |
|---|---|---|
| Candidate home page | Anonymous application availability and response hardening | Pass |
| Liveness | Candidate process health | Pass |
| Signup | Signup rendering and federation-default safety | Pass |
| Shared upload ownership | Anonymous access to an owner-bound upload resume resource | Pass: denied |
| WFL membership | Anonymous attempt to join a WFL session | Pass: denied |
| Restaurant website input | Anonymous submission containing a `javascript:` website URI | Pass: denied |
| Federation disabled | ActivityPub outbox while federation is disabled | Pass: unavailable |
| Production isolation | Existing port 8080 listener during candidate testing | Pass: unchanged |

## Data Sent

- `GET http://127.0.0.1:8081/`
- `GET http://127.0.0.1:8081/actuator/health/liveness`
- `GET http://127.0.0.1:8081/signup`
- `GET http://127.0.0.1:8081/api/shared-folder/2026-07-17/uploads/00000000-0000-0000-0000-000000000001`
- `POST http://127.0.0.1:8081/api/whatsforlunch/restaurant/2026-05-17/sessions/00000000-0000-0000-0000-000000000001/join`
- `POST http://127.0.0.1:8081/api/whatsforlunch/restaurant/2024-12-15`

```json
{"name":"probe","website":"javascript:alert(1)"}
```

- `GET http://127.0.0.1:8081/ap/users/security-probe/outbox`
- No authorization credentials were sent to the three access-control probes. Normal HTTP content negotiation headers were used; no production cookies or tokens were supplied.

## Response Received

- Home: `200 OK`, `Content-Type: text/html`, 4,041-byte body, and `Cache-Control: no-store`.
- Liveness: `200 OK`.
- Signup: `200 OK`; the federation input rendered with `disabled="disabled"` and was not checked.
- Shared upload resume resource: `403 Forbidden`, JSON response, private/no-store caching.
- WFL join: `403 Forbidden`.
- Unsafe restaurant website submission: `403 Forbidden`.
- Federation outbox: `404 Not Found` while federation was disabled.
- Home-page security headers included Content-Security-Policy, `X-Content-Type-Options: nosniff`, `Referrer-Policy: strict-origin-when-cross-origin`, Permissions-Policy, and no-store caching.

## Pass / Fail

All eight runtime cases passed. The candidate started from the exact remediated commit, exposed the expected public pages, denied unauthenticated state-changing and owner-bound operations, kept disabled federation unavailable, rendered signup with a safe federation default, and did not disturb the production listener.

## Evidence

- Strict clean build from a second empty Gradle home: `BUILD SUCCESSFUL in 2m 27s`; 21 tasks executed.
- Java/JUnit XML aggregate: 1,575 tests, 0 failures, 0 errors, 4 skipped across 227 suites.
- Browser JavaScript suite: 279 tests, 279 passed, 0 failed.
- Dependency verification: 395 components and 727 artifacts; every artifact has a valid SHA-256 checksum and no trust bypass was present.
- Gradle wrapper 9.6.1 distribution SHA-256: `9c0f7faeeb306cb14e4279a3e084ca6b596894089a0638e68a07c945a32c9e14`.
- Workflow review: 13 GitHub Action references pinned to immutable commits.
- Final Codex Security report: `C:\Users\Christopher\AppData\Local\Temp\codex-security-scans\christopherbell.dev\5a2186ea5ea2b946faecead2b514f408bab6031e_20260729T170846\report.md`; all 26 review rows closed with zero remaining findings.
- The expected non-elevated ACL denial prevented reading production deployment secrets/configuration. No ACL was weakened and no production secret was copied into the candidate environment.

## Bugs / Follow-ups

No runtime defect or remaining security finding was observed. Production deployment and post-deployment verification remain delivery steps, not test gaps in this candidate report.
