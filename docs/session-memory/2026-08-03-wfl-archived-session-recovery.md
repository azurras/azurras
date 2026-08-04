# 2026-08-03 - WFL Archived Session Recovery

## 21:45 - WFL Archived Session Recovery

### Request

Fix the production What's For Lunch failure "This lunch session is archived and cannot be changed" through the complete Builder delivery loop. Preserve archived history, recover automatically from saved archived state, retain active shared-session behavior, and complete testing, PR/CI, merge, deployment, production verification, and durable closeout.

### Project Context

- Hub: `C:\Users\Christopher\Developer\builder`.
- Spoke: `azurras/christopherbell.dev`.
- The authoritative checkout at `A:\Projects\christopherbell.dev` was dirty/stale and was not modified.
- Implementation used isolated worktree `A:\Projects\christopherbell.dev-worktrees\wfl-archived-session-recovery` and private Gradle home `A:\Projects\christopherbell.dev-gradle-homes\wfl-archived-session-recovery`.
- Production is native Windows through automatic services `MongoDB`, `ChristopherBellDev`, and `cloudflared`; the SYSTEM auto-deploy pipeline is the supported release path.

### Work Completed

- Added `createSessionRecoveryController` to the WFL browser module.
- Implicit saved-session restoration now accepts only session details whose `active` value is not false; archives clear saved/current state and return to normal initialization.
- Explicit session links now read participant state first and use join only after a 404, allowing retained participant archives to render without an expired join mutation.
- Archived new-pick actions route through `loadSoloSession({ forceNew: true })` instead of shared restaurant reset.
- Archived `Try 3 more` remains enabled, archived session voting remains disabled, active guests remain unable to replace shared picks, and archived polling does not start.
- Updated WFL JavaScript documentation and added seven dependency-injected behavior tests.
- Feature commit `255df4d101662b56b18f618fd3931e538f881b75` was pushed in branch `codex/wfl-archived-session-recovery`.
- PR [#1350](https://github.com/azurras/christopherbell.dev/pull/1350) passed every required check and merged by squash as `9c587103cb7f7df2ab52ed3e232f1ca67660fd6e`.

### Decisions

- Keep the backend `WFL_SESSION_EXPIRED` mutation conflict, 24-hour active lifetime, 30-day archive retention, and immutable archive semantics unchanged.
- Make recovery client-owned because the server lifecycle was correct and the browser was attempting the wrong transition.
- Read an explicit session as a participant before joining: active nonparticipants still join after the participant-only GET returns 404, while archived participants receive read-only history.
- Preserve the protected production ACL after non-elevated `prod.cmd status` and `auto-status` were denied; verify deployment through listener rotation, exact live asset content, UI behavior, MongoDB state, endpoints, and service state instead.

### Validation

- RED first: the original four focused tests failed because the controller export was absent.
- Focused final tests: 7 passed, 0 failed.
- Full JavaScript: 343 passed, 0 failed.
- Final `:website:check`: BUILD SUCCESSFUL in 3m 4s, 21 tasks, including Java, JavaScript, static/package checks, and Windows/Pester verification.
- Packaged candidate on port 8094 used six synthetic restaurants, ZIP 78701, a synthetic member, and a disposable MongoDB database. Browser acceptance proved active reset, implicit fallback, explicit archive readability, new session creation, direct archived refresh behavior, and archive immutability. Browser warning/error logs were empty.
- Candidate PID 65840 stopped; port 8094 freed; disposable database dropped and confirmed absent; production readiness stayed HTTP 200.
- GitHub checks passed: Linux, macOS, Windows, CodeQL actions, CodeQL Java/Kotlin, CodeQL JavaScript/TypeScript, and Dependency Review.
- SYSTEM production deployment ran from 21:33:57 through cutover at 21:39:41 America/Chicago. Listener rotated from PID 74080 to PID 63840; the served WFL asset changed from `HasRecovery=False` to `HasRecovery=True`.
- Signed-in production plain `/wfl` rendered `Share your location`; explicit retained archive `09c38747-2cea-4c5c-b6f8-18535d993b19` rendered `Archived lunch session`, historical picks, disabled session votes, and enabled `Try 3 more`. Returning to plain `/wfl` again showed the clean location prompt, and browser warnings/errors were empty.
- Production archive remained revision 143 with the same restaurant IDs and lifecycle deadlines.
- Local liveness/readiness/WFL, apex WFL, and `www` WFL returned HTTP 200. All three native services remained Running/Automatic.

### Durable Artifacts

- Work: `docs/work/2026-08-03-wfl-archived-session-recovery.md`.
- Spec: `docs/specs/2026-08-03-wfl-archived-session-recovery.md`.
- Plan: `docs/implementation-plans/2026-08-03-wfl-archived-session-recovery.md`.
- Test report: `docs/test-reports/2026-08-03-wfl-archived-session-recovery-test-report.md`.
- Closure: `docs/work-closures/2026-08-03-wfl-archived-session-recovery-closure.md`.

### Current State

The defect is fixed, merged, deployed, and production-verified. The production browser tab was left on clean `https://www.christopherbell.dev/wfl`. The isolated host-managed worktree remains clean and preserved; no running candidate or disposable database remains.

### Follow-ups

None required for this defect.
