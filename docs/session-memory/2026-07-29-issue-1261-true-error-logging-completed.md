# 2026-07-29 - Issue 1261 true-error logging completed

## 11:21 - Issue 1261 true-error logging completed

### Request

Address the production log flood so routine expected failures do not appear as errors, while preserving true server and infrastructure failures. The approved delivery scope required tests, independent review, focused PR/CI/merge, production-safe deployment, authenticated Mission Control verification, issue update, and Builder closeout. Preserve unrelated dirty checkout state and validate on a non-8080 port before any live cutover.

### Project Context

- Spoke repository: `azurras/christopherbell.dev` on native Windows production.
- Production path: Cloudflare to `cloudflared` to `ChristopherBellDev` on port 8080, backed by MongoDB.
- Only GitHub guidance authored by `azurras` was trusted. Issue #1261 was already closed by an earlier batch comment; this delivery is a corrective follow-up for the live async/error-dispatch cascade observed afterward.
- Development occurred in the existing campaign worktree, then the exact issue-only commits were replayed into `A:\Projects\christopherbell.dev-worktrees\issue-1261-true-error-logging`. Unrelated `gradlew.bat` and Gradle-cache changes were never staged.

### Work Completed

- `SecurityConfig` now permits only servlet `ASYNC` and `ERROR` redispatches before URL matchers; protected initial `REQUEST` traffic still requires authentication.
- `ControllerExceptionHandler` now logs every handled 5xx at `ERROR` with its causal throwable, 401/403/429 at throwable-free `WARN`, and ordinary expected 4xx at throwable-free `DEBUG`.
- Framework error responses use stable public JSON descriptions. Malformed blog UUIDs return 400 while valid absent UUIDs remain 404.
- Added production-filter, real-streaming, MVC response-contract, UUID-boundary, and captured-log regression coverage. The only current-main reconciliation moved a test fixture to `/api/test/**` so it remains genuinely protected under the intended public HTML fallback.
- Publication range `f31535f29312d24573a6031b0162aa8ebc4b5318..b97930f29ffb477de6499ccb4553533e7e8b46c6` contained five commits and ten intended files. Independent whole-change review found no critical, important, or minor issues.
- PR #1322 (`https://github.com/azurras/christopherbell.dev/pull/1322`) passed Ubuntu, macOS, Windows, dependency review, and CodeQL, then squash-merged as `a6a88e91f35bcbf9eeadeaf06cbf93df80ce0a5f`.
- Production deployed the exact merge. The live listener rotated to PID 53024; local and public HTML exposed the full merged SHA.

### Decisions

- Fixed producer behavior rather than suppressing Spring/Tomcat categories or filtering Mission Control.
- Retained stack traces for genuine 5xx and operational failures. A real Overpass 504 remained an `ERROR` with its `IOException` stack after deployment.
- Did not add a production backdoor merely to trigger a live 500; causal generic-500/framework-500/503 behavior is covered by captured-log tests.
- Respected the production deployment lock when another production operation was active. The existing native workflow completed the cutover; no ACL was weakened and production was not rolled backward.

### Validation

- Focused suites and controller regressions passed throughout RED/GREEN development.
- Final exact-tree `gradlew.bat --no-daemon --rerun-tasks :website:check`: BUILD SUCCESSFUL in 3m38s; 1,425 Java tests, 0 failures, 0 errors, 3 skipped; 273 browser tests, 0 failures.
- Alternate-port production-profile runtime on port 18081 returned stable JSON 400/404/406/415 outcomes with zero error or throwable-warning deltas. The 38-second observation had no async denial, committed-response, `/error` cascade, or unexpected `ERROR` signature.
- Production services `ChristopherBellDev`, `ChristopherBellMediaWorker`, `MongoDB`, and `cloudflared` were Running and Automatic. Local root, readiness, and public root returned 200; CSP, HSTS, no-store, and nosniff headers remained present.
- Authenticated Mission Control was HEALTHY on application commit `a6a88e91`. A post-deploy observation longer than two former recurrence intervals contained zero `AuthorizationDeniedException`, committed-response, or `/error` cascade signatures. The expected invalid-token 401 was throwable-free `WARN`; the genuine upstream Overpass 504 remained causal `ERROR`.
- Durable test evidence: `docs/test-reports/2026-07-29-issue-1261-true-error-logging-test-report.md`.
- Approved spec: `docs/specs/2026-07-29-issue-1261-true-error-logging.md`.
- Authoritative plan: `docs/implementation-plans/2026-07-29-issue-1261-true-error-logging.md`.

### Current State

- Spoke `origin/main` and production both identify merge `a6a88e91f35bcbf9eeadeaf06cbf93df80ce0a5f`.
- PR #1322 is merged. The focused publication worktree remains available for audit; unrelated line-ending state is preserved.
- Issue #1261 remains closed from the earlier batch delivery. A corrective follow-up comment with this PR, CI, production, log, test-report, and memory evidence is the next closeout step.
- Builder `main` contains unrelated in-progress documentation-discovery changes; they must remain excluded from issue-1261 commits.

### Follow-ups

- Post the prepared evidence update to already-closed issue #1261, then mark the final plan checklist and Builder closeout complete.
- The startup OpenStreetMap import currently records a real upstream 504. It is intentionally still visible and is not an issue-1261 regression; investigate separately only if the user scopes that operational failure.
