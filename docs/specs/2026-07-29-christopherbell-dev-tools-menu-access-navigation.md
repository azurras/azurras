# ChristopherBell.dev Tools Menu Access Navigation

## Document Status

ready-for-execution

## Purpose

Make Tools the single navigation home for Music, Command Center, and Back Office while preventing the navigation from advertising protected destinations to accounts that cannot use them.

## Background

The current navigation exposes Music on the top rail and exposes Back Office and Command Center in the authenticated profile menu. Tools already owns public utilities and permission-gated Shared Folder. The current-account API supplies the exact role and permission set needed to derive visibility without a second request.

## Goals

- Remove Music from the top-level navigation.
- Remove Back Office and Command Center from the profile menu.
- Add all three destinations to Tools when the current account has effective access.
- Keep the entire visible Tools list alphabetized.
- Preserve server-side authorization as the authority; navigation visibility is only presentation.

## Non-Goals

- No route, controller, permission, or access-log behavior changes.
- No redesign of the dropdown or mobile navigation.
- No change to the direct Back Office/Command Center cross-links inside their pages.
- No change to public visibility of existing public tools.

## Requirements

1. A signed-out visitor and an authenticated account without Music capability must not see Music in navigation.
2. Effective Music access is true for `ADMIN`, `MUSIC_READ`, or `MUSIC_WRITE`.
3. Back Office and Command Center appear only for exact `ADMIN` role.
4. Shared Folder retains its existing `ADMIN`, `SHARED_FOLDER_READ`, or `SHARED_FOLDER_WRITE` effective-read rule.
5. The final Tools list is sorted alphabetically by displayed label after conditional items are included.
6. Feed, Explore, and Messages remain top-level destinations.
7. Profile and Logout remain in the profile menu; administrative destinations do not.
8. Direct protected routes continue enforcing their existing server-side access rules.

## Proposed Approach

Add a pure effective-read helper beside the existing Music browser helpers. Change `toolsMenuItems` to accept a named access-state object, build the public and conditional destinations, and sort the final list by label. The nav component derives Music and Shared Folder visibility from the same `/api/accounts/me` response, clears both on logout or profile-load failure, and renders administrative items only when the stored current role is `ADMIN`. Remove the obsolete administrative profile-menu helper and Music top-level entry.

This makes the pure Tools-list function the single source of truth for visibility and ordering. Tests assert complete label arrays for signed-out, Music listener, Shared Folder reader, and administrator states, plus the absence of moved entries from their former menus.

## Files and Modules

- `website/src/main/resources/static/js/lib/music.js`: effective Music read helper.
- `website/src/main/resources/static/js/components/nav.js`: access state, menu ownership, rendering, and ordering.
- `website/src/test/js/music.test.js`: effective Music access cases.
- `website/src/test/js/nav-messages-link.test.js`: Tools visibility/order and former-menu removal.

## Validation Plan

- Witness focused JavaScript tests fail before production edits.
- Run focused Music/navigation tests after implementation.
- Run the full `:website:check` task.
- Start the app on a non-production port and inspect rendered navigation for anonymous and available authenticated fixtures without altering production data.
- Require green multi-platform CI and CodeQL before merge.
- Verify the automatic production rollout leaves the public site healthy and serves the changed navigation asset.

## Acceptance Criteria

- Music, Command Center, and Back Office appear only in Tools and only with effective access.
- Every visible Tools list is alphabetized.
- Existing public Tools items and Shared Folder gating are unchanged.
- Automated, local runtime, CI, and production verification pass.

## Open Questions

None. The user's standing authorization applies to this focused design.
