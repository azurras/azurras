# Command Center Reliability Issues 1299-1301 Test Report

## Document Status

complete

## Story/Issue

- `azurras/christopherbell.dev#1299` â€” startup configuration validation
- `azurras/christopherbell.dev#1300` â€” configured power delay and Windows result verification
- `azurras/christopherbell.dev#1301` â€” durable pending machine-power actions
- Pull request: https://github.com/azurras/christopherbell.dev/pull/1328

All issue text and guidance used here came from `azurras`; the issues had no comments or attachments.

## Branch

- Branch: `codex/issues-1299-1301-20260729`
- Pre-squash tip: `461a4326`
- Merged production commit: `044299c8876dc3c421afac191194a8bcdeaa1260`
- PR #1328 squash-merged to `main` on 2026-07-29.

## App / Environment

- Spring Boot website with profile `prod,deploy-smoke`
- Alternate base URL: `http://127.0.0.1:8096`
- Isolated MongoDB database: `cbell_issues_1299_1301_20260729`
- Isolated fixture paths under `build/runtime-8096`
- Production local URL: `http://127.0.0.1:8080`
- Public URLs: `https://christopherbell.dev/` and `https://www.christopherbell.dev/`
- Production services: `MongoDB`, `ChristopherBellDev`, `ChristopherBellMediaWorker`, and `cloudflared`
- Protected values stayed in environment/installed configuration and were not printed.

## Local Run Details

The candidate was built with an isolated Gradle home using `.\gradlew.bat :website:check --console=plain`, then started with:

```powershell
java.exe -jar website\build\libs\website.jar `
  --server.port=8096 `
  --command-center.sensor-libraries-enabled=false
```

The process received the two named profiles, explicit isolated `SPRING_MONGODB_URI` and `SPRING_MONGODB_DATABASE`, disabled mail/federation, a fixture JWT secret, and isolated shared/music paths. Logs were captured under `A:\Projects\christopherbell.dev-worktrees\issues-1299-1301-20260729\build\runtime-8096`. Restarts stopped only the process owning port 8096. The final listener was stopped, port 8096 was free, and the pending-action fixture collection was cleared.

The protected SYSTEM deployment pipeline deployed the merge. The listener rotated to PID 49588 for `044299c8`. Protected deployment configuration/log ACLs denied this non-elevated shell as intended and were not weakened.

## Test Cases

1. Start the packaged production-profile candidate on port 8096.
2. Exercise root, liveness, readiness, and anonymous command-center routes.
3. Seed a future `RESTART_COMPUTER` reservation, restart, and verify it remains.
4. Seed an elapsed `SHUTDOWN_COMPUTER` reservation, restart, and verify reconciliation removes it.
5. Run the full strict Gradle gate and all GitHub checks.
6. Verify exact release SHA, protected access, internal health, public roots, and service startup state in production.

## Data Sent

Unauthenticated requests:

- `GET http://127.0.0.1:8096/`
- `GET http://127.0.0.1:8096/actuator/health/liveness`
- `GET http://127.0.0.1:8096/actuator/health/readiness`
- `GET http://127.0.0.1:8096/api/v1/command-center/snapshot`
- The same local routes on production port 8080
- `GET https://christopherbell.dev/`
- `GET https://www.christopherbell.dev/`

Mongo fixtures used fixed ID `machine-power`, enum actions, and UTC timestamps. One deadline was future and one elapsed. No caller-controlled command, executable, service name, or shell text was supplied.

## Response Received

- Alternate root/liveness/readiness status code: 200.
- Alternate anonymous command-center status code: 403 Forbidden.
- Future reservation response body from `mongoexport`: exactly one `machine-power` document with `RESTART_COMPUTER` after restart.
- Elapsed reservation response body after restart: zero documents; the log excerpt showed the fixed-ID `executeAt <= now` delete query.
- Normal `:website:check`: `BUILD SUCCESSFUL in 4m 22s`; 1,519 Java tests with 0 failures/errors and 3 skips; 289 JavaScript tests with 0 failures/errors; JAR and sensor runtime checks passed.
- PR checks: Windows, Ubuntu, macOS, dependency review, and all CodeQL analyses passed.
- Mainline CI Build and CodeQL for `044299c8`: success.
- Stable production root/liveness/readiness/apex/`www` status code: 200.
- Production anonymous command-center status code: 403 Forbidden.
- Root response body fingerprints: exact SHA `044299c8876dc3c421afac191194a8bcdeaa1260`.
- Four Windows services: Running and Automatic.

## Pass / Fail

- Startup constraints: PASS â€” boundary/cross-field tests reject invalid values and shipped profiles validate.
- Fixed Windows command boundary: PASS â€” configured delay/timeout, exact arguments, zero exit, timeout, and nonzero behavior passed injected-runner tests.
- Durability/reconciliation: PASS â€” real restart preserved future state and removed elapsed state.
- Access boundary: PASS â€” anonymous command-center requests returned 403.
- Local/platform validation: PASS â€” full local gate, PR matrix, dependency review, PR CodeQL, mainline CI, and mainline CodeQL passed.
- Production acceptance: PASS â€” exact fingerprint, health, public URLs, and startup services passed.

## Evidence

- Spec: `docs/specs/2026-07-29-command-center-configuration-and-durable-power-actions.md`
- Plan: `docs/implementation-plans/2026-07-29-command-center-configuration-and-durable-power-actions.md`
- PR: https://github.com/azurras/christopherbell.dev/pull/1328
- Merge: `044299c8876dc3c421afac191194a8bcdeaa1260`
- Runtime logs: `A:\Projects\christopherbell.dev-worktrees\issues-1299-1301-20260729\build\runtime-8096`
- Mainline CI: https://github.com/azurras/christopherbell.dev/actions/runs/30489529812
- Mainline CodeQL: https://github.com/azurras/christopherbell.dev/actions/runs/30489529814
- Stable production acceptance: 2026-07-29T15:50:42-05:00

## Bugs / Follow-ups

The rebase exposed missing SHA-256 entries from the preceding dependency bump; PR #1328 added only the generated OSHI 7.4.2 and Bootstrap 5.3.8 entries, and strict verification/dependency review passed. Production readiness briefly reported `OUT_OF_SERVICE` during the protected post-cutover transition, then returned 200 on the same PID while MongoDB stayed reachable; the complete stable acceptance set passed afterward.

No live restart/shutdown was executed because it would disrupt this production host. The injected-runner tests cover the exact fixed commands and result handling. No unresolved product gap remains for #1299-#1301.
