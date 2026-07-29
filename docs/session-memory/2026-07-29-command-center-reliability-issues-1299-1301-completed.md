# 2026-07-29 - Command Center Reliability Issues 1299 1301 Completed

## 15:53 - Command Center Reliability Issues 1299 1301 Completed

### Request

Continue the approved autonomous campaign to address every open GitHub issue for `azurras/christopherbell.dev`. Complete issues #1299-#1301 through specification, implementation, tests, PR/CI, merge, production verification, issue closure, and Builder evidence without asking for routine approval. Preserve the dirty authoritative website checkout and validate on a non-8080 port before production cutover.

### Project Context

Builder is the workflow hub at `C:\Users\Christopher\Developer\builder`. Website work used isolated worktree `A:\Projects\christopherbell.dev-worktrees\issues-1299-1301-20260729` on branch `codex/issues-1299-1301-20260729`; the dirty authoritative `A:\Projects\christopherbell.dev` checkout was not changed. Only issue content from trusted author `azurras` was used; #1299-#1301 had no comments or attachments. Production is the native Windows host and deploys through the protected SYSTEM pipeline, with candidate validation before the port-8080 switch.

### Work Completed

- Saved and pushed Builder spec `docs/specs/2026-07-29-command-center-configuration-and-durable-power-actions.md` (`7984a6a`).
- Saved, validated, reviewed, and pushed plan `docs/implementation-plans/2026-07-29-command-center-configuration-and-durable-power-actions.md` (`8e9185a`).
- Added startup validation to `CommandCenterProperties` for scalar, nested, cross-field, finite-threshold, challenge-capacity, whole-second delay, and host-independent Windows absolute-path invariants. Shipped local and production profiles validate.
- Changed `WindowsCommandExecutor` to use the configured power delay and result timeout, keep fixed enum-only argument arrays, wait for completion, require exit code zero, forcibly terminate timeouts, and preserve interruption.
- Replaced process-local pending state with a fixed-ID Mongo boundary (`PendingActionStore`, `PendingActionDocument`, `MongoPendingActionStore`) and an application-readiness reconciler. Reservation is atomic; rollback and clear use exact identity; future state survives restart; elapsed state is cleared; repeated cancellation is an audited no-op after the first successful cancel.
- Made challenge capacity configurable and enforced per-actor and total limits.
- Rebase onto main exposed that dependency PR #1310 upgraded OSHI and Bootstrap without verification metadata. Added only generated SHA-256 entries for OSHI 7.4.2 and Bootstrap 5.3.8 so strict verification works.
- Website commits before squash: `083c1d9a`, `d472b82d`, `d916cd99`, `461a4326`.
- Opened PR #1328, changed closure keywords to `Addresses` so issues remained open through deployment, and squash-merged it as `044299c8876dc3c421afac191194a8bcdeaa1260`.
- Saved and pushed runtime/production report `docs/test-reports/2026-07-29-command-center-reliability-issues-1299-1301-test-report.md` (`167d458`).

### Decisions

- Used MongoDB rather than an in-memory or file-backed reservation because the app already relies on Mongo and atomic fixed-key conditional updates provide a clear single-pending-action invariant across process restarts.
- Kept the operating-system boundary enum-only; browser input cannot supply executable paths, service names, or arbitrary arguments.
- Reserved durable state before audit/launch and rolled it back on audit or launch failure so accepted UI state never survives a rejected action.
- Used a dedicated readiness listener rather than placing lifecycle behavior on the action service, avoiding controller-slice lifecycle interactions and making reconciliation independently testable.
- Did not execute a real restart/shutdown on this production host. Injected-runner tests prove exact argument, delay, timeout, nonzero, and completion behavior without disrupting production.
- Did not weaken protected ProgramData ACLs when the non-elevated shell was denied; monitored the authorized SYSTEM deployment through listener/fingerprint/health evidence instead.

### Validation

- TDD red/green evidence covered new properties, Windows command result handling, persistent state, restart recreation, rollback, idempotent cancel, and readiness lifecycle registration.
- Normal strict `:website:check` on the final branch: BUILD SUCCESSFUL in 4m 22s; 1,519 Java tests, 0 failures/errors, 3 skipped; 289 JavaScript tests, 0 failures/errors; boot JAR and sensor runtime checks passed.
- Packaged local `prod,deploy-smoke` run on 8096: root/liveness/readiness 200 and anonymous command-center snapshot 403.
- Real isolated Mongo restart: future `RESTART_COMPUTER` document survived; elapsed `SHUTDOWN_COMPUTER` document was deleted during ApplicationReadyEvent; test listener stopped and fixture collection cleared.
- PR #1328: Windows/Linux/macOS CI, dependency review, and CodeQL actions/Java/JavaScript all passed.
- Mainline CI Build run 30489529812 and CodeQL run 30489529814 succeeded for merge SHA.
- Production listener rotated to PID 49588. Stable acceptance at 2026-07-29T15:50:42-05:00 showed exact SHA on local/apex/www roots, root/liveness/readiness 200, anonymous command center 403, and MongoDB/ChristopherBellDev/ChristopherBellMediaWorker/cloudflared Running and Automatic.
- Readiness briefly reported OUT_OF_SERVICE during post-cutover transition, then recovered to 200 on the same PID while MongoDB remained reachable; the complete stable acceptance set passed afterward.

### Current State

- PR #1328 is merged. Issues #1299-#1301 intentionally remain open until evidence comments are posted immediately after this memory checkpoint.
- The isolated issue worktree remains because the remote branch was deleted after merge. Its only worktree modification is the repository-known `gradlew.bat` clean-filter artifact; it was never staged or committed.
- Production serves `044299c8876dc3c421afac191194a8bcdeaa1260` on PID 49588.
- Builder was clean before saving this entry and remains on `main`.

### Follow-ups

- Post closure evidence and close #1299, #1300, and #1301.
- Continue the same autonomous campaign with open issues #1302-#1305: deterministic artifact versions, offline/cached sensor packaging, Windows Pester CI, and CI concurrency/timeouts.
- Preserve the dirty authoritative checkout and use a fresh isolated worktree from current `origin/main` for the remaining build/CI batch.
