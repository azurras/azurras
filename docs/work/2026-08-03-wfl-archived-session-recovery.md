# WFL Archived Session Recovery

## Status

closed

## Objective

Stop the What's For Lunch browser from treating archived shared sessions as mutable current sessions. Automatically discard archived sessions restored from saved browser state, and make a request for new picks from an explicitly opened archived session leave the archive and start a fresh flow without changing archived history.

## Owner and Context

- Hub: `C:\Users\Christopher\Developer\builder`
- Spoke: `A:\Projects\christopherbell.dev`
- Reported by: direct user report on 2026-08-03 after the thumbs-voting production release
- Delivery model: root-cause-first diagnosis, approved narrow design, durable spec/plan, isolated worktree from refreshed `origin/main`, test-first JavaScript fix, alternate-port runtime/browser validation, PR/CI/squash merge, protected production deployment, and final Builder closure

## Root Cause Evidence

- Production retains one archived WFL session and one active WFL session for the reporting account.
- Archived sessions remain readable for 30 days after their 24-hour active window, by design.
- `loadStoredMemberSession()` currently accepts an API detail with `active: false`, keeps it in `activeSession`, and returns success instead of clearing the saved session ID.
- `loadNearbyPicks()` treats any non-null `activeSession` as mutable and calls the shared-session restaurant-reset endpoint.
- The backend correctly classifies that request as `WFL_SESSION_EXPIRED` and returns HTTP 409 with “This lunch session is archived and cannot be changed.”

## Approved Design

- A session restored implicitly from the per-account saved browser key is reusable only when `active !== false`.
- An archived saved session is cleared and normal solo/new-session initialization continues.
- An archived session opened explicitly by URL remains readable and retains its historical picks, votes, link, and archive presentation.
- When the user requests new picks, applies filters, or changes location while viewing an explicit archive, the browser clears the archived client context and starts a new solo flow. It never sends a mutation for the archived session.
- The server archive lifetime, immutable expired-session mutations, HTTP 409 contract, and automatic deletion remain unchanged.

## Related Artifacts

- Specification: [WFL Archived Session Recovery](../specs/2026-08-03-wfl-archived-session-recovery.md), complete
- Implementation plan: [WFL Archived Session Recovery](../implementation-plans/2026-08-03-wfl-archived-session-recovery.md), complete
- Test report: [WFL Archived Session Recovery Test Report](../test-reports/2026-08-03-wfl-archived-session-recovery-test-report.md), complete
- Closure: [WFL Archived Session Recovery Closure](../work-closures/2026-08-03-wfl-archived-session-recovery-closure.md)
- Session memory: [WFL Archived Session Recovery](../session-memory/2026-08-03-wfl-archived-session-recovery.md)

## Spoke Repositories

- `azurras/christopherbell.dev` at `A:\Projects\christopherbell.dev`

## Current State

PR #1350 merged as `9c587103cb7f7df2ab52ed3e232f1ca67660fd6e` after every required CI, CodeQL, and dependency gate passed. The native SYSTEM deployment rotated production to the merged release. Signed-in production verification proved implicit archive fallback, explicit read-only archive rendering, enabled fresh-pick recovery, unchanged archived MongoDB state, zero browser warnings/errors, HTTP 200 local/apex/www WFL responses, and Running/Automatic native services. No required action remains.

## Blockers

None.

## Validation

- Live MongoDB session inventory was inspected read-only at 2026-08-04T00:25Z.
- Current client and server session paths were traced from saved browser restoration through the expired mutation response.
- No protected production ACLs or session data were modified.

## Next Steps

None. This work is closed.
