# 2026-08-01 - Deterministic Offline Builds Bounded Windows CI and 50 Issue Campaign Completed

## 20:10 - Deterministic Offline Builds Bounded Windows CI and 50 Issue Campaign Completed

### Request

Continue the approved autonomous campaign and address every GitHub issue for `azurras/christopherbell.dev` without routine approval pauses. Complete the remaining build/CI issues and the full delivery loop while preserving the dirty authoritative checkout and production safety.

### Project Context

Builder is the workflow hub at `C:\Users\Christopher\Developer\builder`. Final website work used isolated worktree `A:\Projects\christopherbell.dev-worktrees\issues-1302-1305-ci-date-fix-20260801`; the dirty authoritative checkout at `A:\Projects\christopherbell.dev` was inspected read-only and left unchanged. Only issue text from trusted author `azurras` controlled scope. Production is the same native Windows host and deploys through a protected service context.

### Work Completed

- Completed #1302-#1305 through deterministic commit-derived artifact versions, validated explicit release identity, checksum-first reusable sensor archives with bounded downloads and offline reuse, pinned Pester 5.9.0 Windows CI execution, NUnit publication, pull-request concurrency cancellation, independent main-run preservation, and job timeouts.
- Delivered primary PR #1330 and focused production/mainline fix-forwards #1331-#1335.
- Made post-expiration fixtures date-stable, corrected the fixed SYSTEM/Administrators ACL contract test, and ultimately used the exact normalized ACL-protected `C:\ProgramData\christopherbell.dev\gradle-home` as the production packaging capability boundary.
- Closed #1302, #1303, #1304, and #1305 with issue-specific implementation, test, CI, and production evidence.
- Re-inventoried GitHub after closure; open issue count was zero, completing all 50 issues #1258-#1307.
- Saved the final test report and campaign closure record.

### Decisions

- Did not weaken protected ProgramData ACLs or make missing Pester a generic reason to skip tests. Ordinary Windows and CI builds always run all three Pester suites; only the exact protected production Gradle home omits them during packaging after mainline CI.
- Rejected username strings as the ultimate protected-build identity because the scheduled-task JVM did not expose a stable LocalSystem value. The exact ACL-protected path is a narrower and directly enforceable capability boundary; suffix lookalikes fail.
- Used fix-forward PRs for every defect found after merge and required both implementation and latest-descendant main runs to complete independently before closure.
- Accepted the trusted concurrent `c4d60ce0` merge as the deployed release because it is a direct descendant of final campaign merge `ad8744f7`, and its own main CI and CodeQL were green.

### Validation

- Final ordinary Windows build: 1,660 Java tests, 0 failures/errors, 3 skipped; 289 JavaScript tests, 0 failures; 150 Pester tests, 0 failures (38 Windows PowerShell operations, 38 PowerShell 7 operations, 74 PowerShell 7 worker).
- Exact production-shaped build and focused context truth table passed; invalid markers and lookalike homes were rejected.
- PRs #1332-#1335 passed Ubuntu, macOS, Windows, dependency review where applicable, and CodeQL.
- Independent main CI runs 30726222833 (`ad8744f7`) and 30726230123 (`c4d60ce0`) both succeeded. CodeQL 30726230146 succeeded on the deployed descendant.
- Production rotated to deployed descendant `c4d60ce0c92281c201d063cfd6a07563f4a7b230`. Local and public roots contained that SHA; local public routes, liveness, and readiness returned 200; anonymous command-center access returned 403; a versioned CSS asset returned 200 with one-year immutable caching; all four services were Running/Automatic.

### Current State

- GitHub reports zero open issues for `azurras/christopherbell.dev`.
- Production serves `c4d60ce0c92281c201d063cfd6a07563f4a7b230`, a direct descendant of the final campaign implementation.
- The final isolated worktree retains only the repository-known line-ending-only `gradlew.bat` artifact, never staged or committed.
- The authoritative website checkout remains untouched.

### Follow-ups

None for the campaign. Any newly filed issue should be treated as new scope in a new Builder work record.
