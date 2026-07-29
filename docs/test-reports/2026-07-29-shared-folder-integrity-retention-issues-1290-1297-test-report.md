# Shared-Folder Integrity and Retention Issues 1290-1297 Test Report

## Document Status

complete

## Story/Issue

GitHub issues `azurras/christopherbell.dev` #1290 through #1297 and pull request #1326.

## Branch

`codex/issues-1290-1297-20260729`, source commit `b93076780c9ff2925d98e650f4e4a6efc3148b24`, based on `b28031d535effef1fcbd547ba8f7dffdd4e76193`; squash-merged as `f67c90eed9b29215d562b2ac3670528f614508e9`.

## App / Environment

The exact Spring Boot website JAR ran with profile `local` at `http://127.0.0.1:8095`, using disposable database `cbell_issue_1290_1297_20260729` and disposable shared/system roots below the isolated worktree build directory. Both Mongo URI and database name were explicitly overridden, mail was disabled, the shared-folder feature was enabled, and a local-only JWT secret was supplied. Production remained on port 8080 and database `christopherbell` during local acceptance.

## Local Run Details

The final checked JAR was built by `./gradlew.bat :website:check --no-daemon --console=plain` and launched as hidden PID `57180` with `java -jar website\build\libs\website.jar`. Cold startup applied migration 012 and populated the disposable restaurant catalog before readiness returned 200. The runtime was exercised through real HTTP and MongoDB boundaries. PID 57180 was stopped, port 8095 was confirmed free, the disposable database was dropped and confirmed absent, and the generated runtime tree was removed by the project clean task. Production PID 52804 stayed healthy until the guarded post-merge rollout.

## Test Cases

| Case | Expected | Result |
|---|---|---|
| Complete gate | Java, JavaScript, packaging, sensor, and policy checks pass | Pass |
| Cold catalog | First search reports BUILDING, then publishes FRESH | Pass |
| Stable search pages | Two stable pages share a generation with no duplicates | Pass |
| Mutation invalidation | Create advances generation and invalidates old cursor | Pass |
| Authorization | Anonymous shared-folder APIs remain forbidden | Pass |
| Upload retention | Completed upload receives a seven-day TTL | Pass |
| Download audit | Terminal audit records actual served bytes | Pass |
| Migration/indexes | Migration 012 and cleanup/TTL indexes apply | Pass |
| Production rollout | Exact merged release and invariants verify live | Pass |
| Cleanup | Candidate resources are removed | Pass |

## Data Sent

- Created one disposable active USER through the real CSRF-protected signup flow and granted `SHARED_FOLDER_READ` and `SHARED_FOLDER_WRITE` only in the disposable database.
- Added `Alpha/report.txt`, `Beta/report.txt`, and `Gamma/report.txt` under the disposable shared root.
- Sent authenticated `GET /api/shared-folder/2026-07-17/search?query=report&size=2`, followed its opaque cursor, then retried that cursor after authenticated `POST /api/shared-folder/2026-07-17/folders` for `Mutation-visible`.
- Sent a four-byte upload (`data`) through create, chunk append with base64url SHA-256, and complete endpoints; then downloaded `runtime-upload.txt`.
- Queried migration records, upload/media indexes, completion retention, terminal cleanup coverage, active-work TTL exclusions, and radio version state.
- Requested local `/`, `/shared`, liveness, readiness, protected anonymous routes, and public `https://christopherbell.dev/` after deployment.

## Response Received

- Local root and shared page returned HTTP status code: 200. Readiness returned HTTP status code: 200 with body `{"status":"UP"}`.
- Cold search first returned generation 1 with `BUILDING`; the next response returned `FRESH`, paths `Alpha/report.txt,Beta/report.txt`, and a cursor. Page two returned `Gamma/report.txt` with the same generation and no cursor.
- Folder creation returned HTTP status code: 201. The old cursor returned HTTP status code: 409. Fresh search returned generation 2 and `Mutation-visible`.
- Upload progressed from ACTIVE offset 0 to ACTIVE offset 4 to COMPLETED. MongoDB stored `deleteAt=2026-08-05T19:20:14.853Z`.
- Download returned HTTP status code: 200 and four bytes. Audit stored `DOWNLOAD_STARTED size=4` and `DOWNLOAD_COMPLETED size=4` with accepted outcomes.
- Migration `012-retain-shared-folder-work` was APPLIED with checksum `20b57d9152f4ea8b78f3de3b808325d195a9a864a5ace222c66954066228396a`; required cleanup and TTL indexes matched their expected keys.
- After merge, the listener rotated to PID 54448. Local root, shared page, liveness, readiness, and public root each returned HTTP status code: 200. Root assets used merge fingerprint `f67c90eed9b29215d562b2ac3670528f614508e9`.
- Production counts were zero for completed uploads missing `deleteAt`, non-completed uploads with `deleteAt`, terminal media missing cleanup scheduling, and active media with `deleteAt`. The radio document has optimistic `version: 0`.

## Pass / Fail

PASS. Final `:website:check` completed 1,507 Java tests with 0 failures/errors and 3 skipped; JavaScript, boot JAR, sensor-runtime, and policy gates passed. PR #1326 and post-merge main passed Ubuntu, macOS, Windows, dependency review, and CodeQL. All runtime and production checks passed.

## Evidence

- Pull request: https://github.com/azurras/christopherbell.dev/pull/1326
- PR CI: https://github.com/azurras/christopherbell.dev/actions/runs/30484155028
- PR CodeQL: https://github.com/azurras/christopherbell.dev/actions/runs/30484154561
- Post-merge CI: https://github.com/azurras/christopherbell.dev/actions/runs/30484590547
- Post-merge CodeQL: https://github.com/azurras/christopherbell.dev/actions/runs/30484585859
- Production services `ChristopherBellDev`, `MongoDB`, and `Cloudflared` were Running with Automatic startup.
- Anonymous production search, radio, and content returned HTTP status code: 403. The deployed shared page contains Load more and cursor-aware JavaScript.

## Bugs / Follow-ups

No unresolved defect or verification gap remains for #1290-#1297. Continue with the remaining open website issues beginning at #1299.
