# WFL Archived Session Recovery Closure

## Final Status

closed

## Source

Direct user-reported production bug on 2026-08-03: the What's For Lunch page showed "This lunch session is archived and cannot be changed." No separate GitHub issue existed, so no external issue close operation was applicable. PR #1350 had no comments, reviews, or attachments; therefore no untrusted GitHub input influenced scope or closure.

## Completed Scope

- Rejected archived saved sessions during implicit member-session restoration and returned the browser to normal location/picker initialization.
- Loaded explicit archive URLs by participant read first, joining only when a nonparticipant active link returns 404.
- Kept explicit archives readable with historical picks and disabled session votes.
- Routed refresh, filters, and location actions from archives into a fresh picker instead of an expired-session mutation.
- Preserved active host refreshes, active guest restrictions, server 409 expiry-race enforcement, retention, voting, favorites, and weighted selection.

## Spoke Repository

- Repository: `azurras/christopherbell.dev`
- Isolated worktree: `A:\Projects\christopherbell.dev-worktrees\wfl-archived-session-recovery`
- Feature commit: `255df4d101662b56b18f618fd3931e538f881b75`
- Pull request: [#1350 Recover archived WFL sessions locally](https://github.com/azurras/christopherbell.dev/pull/1350)
- Merge commit: `9c587103cb7f7df2ab52ed3e232f1ca67660fd6e`
- Merge state: merged to `main` by squash after all required checks passed

## Automated Validation

- Focused Node recovery suite: 7 passed, 0 failed.
- Full JavaScript suite: 343 passed, 0 failed.
- `:website:check`: BUILD SUCCESSFUL in 3m 4s; 21 tasks, including Java, JavaScript, static/package checks, and Windows/Pester verification.
- GitHub: Linux, macOS, Windows, CodeQL actions, CodeQL Java/Kotlin, CodeQL JavaScript/TypeScript, and Dependency Review all passed.

## Local Runtime Validation

The packaged candidate ran on `http://127.0.0.1:8094` against disposable MongoDB fixtures. Browser acceptance proved active shared reset, implicit archive fallback, explicit archive readability, new-session creation from an archive, direct archived `Try 3 more`, archive immutability, and zero browser warnings/errors. The candidate stopped, port 8094 was freed, and the disposable database was dropped. See [WFL Archived Session Recovery Test Report](../test-reports/2026-08-03-wfl-archived-session-recovery-test-report.md).

## Production Deployment and Acceptance

- The native SYSTEM automatic deployment started at 21:33:57 and completed listener rotation at 21:39:41 on 2026-08-03 America/Chicago.
- Production listener changed from PID `74080` to PID `63840`.
- The live `/js/whats-for-lunch.js` changed from `HasRecovery=False` to `HasRecovery=True`.
- Signed-in plain `https://www.christopherbell.dev/wfl` rendered `Share your location` instead of restoring the retained archive or displaying a 409 error.
- Explicit archive `09c38747-2cea-4c5c-b6f8-18535d993b19` rendered `Archived lunch session`, the three historical restaurants, disabled session vote controls, and enabled `Try 3 more`.
- After read-only verification, the archive remained revision `143` with the same three restaurant IDs and unchanged lifecycle deadlines.
- The Chrome tab was returned to plain `/wfl`, which again rendered the location prompt; browser warnings/errors were empty.
- Local liveness, local readiness, local WFL, apex WFL, and `www` WFL all returned HTTP 200.
- `MongoDB`, `ChristopherBellDev`, and `cloudflared` remained Running/Automatic.
- Protected production config remained ACL-denied to the non-elevated shell; no ACL was weakened.

## Decisions

The server conflict and archive lifecycle were retained because they are correct. Recovery is client-owned: implicit archives are discarded locally, explicit participant archives are read before any join attempt, and fresh selections never mutate known archives.

## Known Gaps and Follow-ups

None for this defect. The host-managed worktree is preserved; it can be removed later through the normal workspace lifecycle after no further PR iteration is needed.

## Closure Readiness

ready

## Closure Text

Resolved and deployed. PR #1350 merged as `9c587103`. All local and GitHub checks passed. Production rotated to the merged release, plain signed-in `/wfl` now recovers from saved archived state, the retained archive remains explicitly readable and immutable, every checked endpoint is HTTP 200, and all native production services remain Running/Automatic. No separate GitHub issue existed to close and no follow-up gap remains.
