# WFL Archived Session Recovery

## Status

active

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

- Specification: [WFL Archived Session Recovery](../specs/2026-08-03-wfl-archived-session-recovery.md), ready for execution
- Implementation plan: [WFL Archived Session Recovery](../implementation-plans/2026-08-03-wfl-archived-session-recovery.md), ready for execution
- Test report: pending implementation
- Closure/session memory: pending final delivery

## Spoke Repositories

- `azurras/christopherbell.dev` at `A:\Projects\christopherbell.dev`

## Current State

The root cause is confirmed in current production data and the merged browser flow. No production data or code has been changed. The user approved the design and written specification; the implementation plan maps the narrow JavaScript controller, RED/GREEN tests, documentation, runtime acceptance, publishing, deployment, and closeout to exact current-main ranges.

## Blockers

None. Implementation execution-mode selection is the next gate.

## Validation

- Live MongoDB session inventory was inspected read-only at 2026-08-04T00:25Z.
- Current client and server session paths were traced from saved browser restoration through the expired mutation response.
- No protected production ACLs or session data were modified.

## Next Steps

1. Validate, review, commit, and push the implementation plan.
2. Select inline or subagent-driven execution.
3. Implement in a clean isolated worktree using failing JavaScript behavior tests first.
4. Complete local runtime/browser validation, PR/CI/merge, protected production deployment, and Builder closeout.
