# WFL Archived Session Recovery

## Document Status

in-progress

## Purpose

Prevent the What's For Lunch browser from restoring an archived shared session as the caller's current mutable session. Preserve archived sessions as readable history while making requests for new picks leave the archived context and begin a fresh flow instead of sending a mutation that the server must reject.

## Background

WFL shared sessions are mutable for 24 hours and remain readable for 30 additional days before MongoDB TTL deletion. The browser stores the last member session ID under an account-scoped local key. On a normal `/wfl` visit, `loadStoredMemberSession()` fetches that ID and currently treats any successful session response as reusable, including details with `active: false`.

The page therefore sets an archived detail as `activeSession`. Later, `loadNearbyPicks()` branches on the presence of `activeSession` rather than its active state and calls the shared-session restaurant-reset endpoint. The mutation store correctly rejects the expired session, and the API returns HTTP 409 with code `WFL_SESSION_EXPIRED` and description “This lunch session is archived and cannot be changed.”

Production evidence on 2026-08-03 confirmed one retained archived session and one active session. The server lifecycle, immutable archive, and conflict response are correct; the client recovery path is wrong.

## Goals

1. Never reuse an archived session restored implicitly from saved browser state.
2. Preserve explicit archived-session links as readable historical views.
3. Turn a request for new picks from an archived view into a fresh solo/new-session flow.
4. Guarantee the browser does not send join, vote, or restaurant-reset mutations for a session detail already known to be archived.
5. Preserve active shared-session behavior, archive retention, server conflict semantics, voting, favorites, filters, location, and weighted restaurant selection.

## Non-Goals

- Do not reactivate, extend, delete early, or modify archived server documents.
- Do not change the 24-hour active lifetime or 30-day archive lifetime.
- Do not weaken the backend `WFL_SESSION_EXPIRED` conflict or mutation-store time predicate.
- Do not migrate production session data.
- Do not redesign the WFL control panel, session sharing, or history UI.
- Do not change restaurant thumb voting or selection weights.

## Requirements

### Implicit saved-session restoration

- `loadStoredMemberSession()` must consider a fetched session reusable only when its public detail is active.
- When the fetched detail has `active: false`, the browser must clear the account-scoped saved session ID, clear the client `activeSession`, stop session polling, and return control to normal `/wfl` initialization.
- Normal initialization must then show the location prompt or load fresh picks using the caller's available location/ZIP state. It must not render the archived session as the current default.
- Missing, forbidden, or otherwise failed stored-session fetches must retain the existing clear-and-fallback behavior.

### Explicit archived-session links

- Navigating to `/wfl?session=<archived-id>` must continue to load and render the archived session read-only when the caller is a participant.
- The archived heading, historical restaurants, votes, participants, link, and `active: false` state must remain visible.
- Session vote controls must remain disabled and no automatic mutation may be attempted.

### Starting fresh from an archive

- When a user requests new picks through refresh, applying filters, or changing location while `activeSession.active === false`, the browser must first leave the archived client context.
- Leaving the archive must clear the saved member-session ID, clear `activeSession`, stop polling, and remove the `session` query parameter.
- The request must continue through the normal solo picker using the selected filters and current ZIP/location behavior.
- If the signed-in picker returns exactly three restaurants, the existing behavior may create and save a new active shared session; it must never update the archived document.

### Active-session compatibility

- When `activeSession.active !== false`, refresh/apply-filter behavior must continue to call the existing host-only shared-session restaurant reset with the current revision.
- Active session polling, votes, favorites, share links, and conflict handling must remain unchanged.

### Failure behavior

- Genuine server 409 responses remain visible when an active-looking client races with expiry or another mutation.
- Location denial, empty results, authentication failures, and unexpected network/server errors must continue through existing error rendering.
- Recovery must not silently swallow unexpected fetch failures beyond the existing stored-session fallback contract.

## Proposed Approach

Create one small browser-owned state transition that leaves an archived session without touching the server. Use it in two places:

1. After an implicitly restored session is fetched, reject `active: false` as a reusable saved session and fall back to normal initialization.
2. Before the nearby-picks branch, detect `activeSession.active === false`, run the local archive-exit transition, and continue into the solo picker rather than the shared-session reset.

Keep explicit URL loading unchanged so archives remain readable. Keep all server code unchanged because it already enforces the required immutable lifecycle.

The implementation should expose the smallest testable JavaScript boundary needed to prove the transition and orchestration. It must not add configuration knobs or broaden the API.

## Files and Modules

- `website/src/main/resources/static/js/whats-for-lunch.js`: saved-session restoration, archive-exit state transition, and nearby-picks orchestration.
- `website/src/test/js/whats-for-lunch.test.js`: regression coverage using the real exported browser orchestration boundary and complete session detail fixtures.
- `website/src/main/resources/static/js/README.md` or the WFL feature README only if the existing documented session lifecycle requires a clarification.
- Server session services, mutation store, repository, model, and retention configuration should remain unchanged.

## Validation Plan

### Automated RED/GREEN evidence

- Add a failing JavaScript test proving an archived saved session is not retained as the current session and its saved ID is cleared.
- Add a failing JavaScript test proving a new-picks request from an explicit archived detail runs the solo picker and never invokes the shared-session reset request.
- Preserve a passing test proving an active session still uses the shared-session reset path.
- Run the focused Node test file, `node --check` on the touched module, full `:website:jsTest`, and full `:website:check` using a private Gradle home.

### Runtime evidence

- Start the packaged merged candidate on a non-8080 port against a disposable database with one active and one archived WFL session.
- In an authenticated browser, prove a normal `/wfl` visit does not restore the archived saved session.
- Open an explicit archived link and prove it remains readable.
- Apply filters or request new picks and prove the browser leaves the archive, obtains fresh picks, and does not receive `WFL_SESSION_EXPIRED`.
- Verify active shared-session refresh still mutates the current session successfully.
- Confirm browser console errors/warnings are empty and production port 8080 remains untouched until merge.

### Publication and production

- Open a ready PR, pass Linux/macOS/Windows CI, Dependency Review, and all CodeQL analyzers, then squash merge.
- Deploy through the protected native Windows path.
- Verify listener rotation, readiness/liveness `UP`, exact current asset version, live `/wfl` recovery in a real browser, service state, and no new archived-session mutation error during the acceptance window.

## Rollback and Recovery

The change is browser-only and does not alter MongoDB data or server contracts. Before production deployment, rollback is the normal branch/PR process. After deployment, the previous application release remains compatible with the unchanged session schema, so normal application rollback remains available if necessary.

## Risks

- Clearing any inactive detail too broadly could hide an explicitly opened archive. Mitigation: apply automatic discard only to the implicit stored-session path; explicit URL loading remains unchanged.
- Falling through with stale `activeSession` could still route into reset. Mitigation: make archive exit clear state before branch selection and prove the reset dependency is never called.
- A session may expire after a client check. Mitigation: retain the backend time predicate and visible HTTP 409 race handling.
- Tests that assert only helpers could miss orchestration. Mitigation: exercise the real exported flow boundary with dependency seams and assert observable calls/state.

## Acceptance Criteria

- A normal `/wfl` visit never restores an archived saved session as current.
- An explicit archived URL remains readable and immutable.
- Refresh/apply filters/location from an archive starts a fresh flow without an expired-session mutation request or error.
- Active session reset behavior remains unchanged.
- Focused, full JavaScript, aggregate repository, alternate-port browser, CI, and production verification all pass.

## Open Questions

None. The user approved the automatic recovery approach on 2026-08-03.
