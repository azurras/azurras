# ChristopherBell.dev Music Catalog Pagination

## Document Status

ready-for-execution

## Purpose

Remove the hidden 100-track Music catalog ceiling and let listeners browse every indexed track through explicit, stable server-backed pages without changing the global radio candidate pool.

## Background

Production contains 1,549 files with supported audio extensions below `A:\Shared\Music`, while the Music catalog request defaults to 100 results and clamps every request to at most 100. The current browser renders only that response and therefore reports 98 ready tracks rather than the full indexed-library count. Favorites and playlists are also filtered from only the loaded response.

The radio uses a separate `radioCandidates(10_000)` query over all present, ready, non-excluded tracks. Catalog page state must never narrow or replace that radio query.

## Goals

- Return a real total for the full filtered catalog.
- Display 50 tracks per numbered page with Previous, Next, and nearby page controls.
- Apply text search and artist, album, and genre filters before counting and paging.
- Keep ordering stable across page requests by adding a unique final sort key.
- Page Favorites and playlist contents rather than filtering only the current All Music page.
- Preserve radio selection from every eligible indexed track, independent of the visible page.

## Non-Goals

- No infinite scrolling or client-side download of the entire catalog.
- No change to playback, download authorization, metadata editing, queue semantics, radio weighting, or the 10,000-track radio safety bound.
- No pagination requirement for the bounded radio-history panel.

## Requirements

- Catalog requests accept zero-based `page` and bounded `size`; the UI uses size 50.
- Invalid negative pages or out-of-range sizes are normalized or rejected consistently at the typed query boundary.
- Catalog responses include `page`, `size`, `totalTracks`, and `totalPages` alongside tracks and facets.
- The default request returns page zero, not a silent unlabelled subset.
- Search/filter changes reset to page zero.
- Moving pages preserves active text and facet filters and does not restart or replace the persistent media player.
- Facet options describe the full applicable result set rather than only the visible page.
- The current page is clamped to the last available page when mutations reduce the result count.
- Page controls are keyboard accessible, indicate the active page, and remain usable on mobile.
- Protected catalog authorization and no-store response headers remain unchanged.
- Radio continues querying the complete eligible catalog through `radioCandidates(10_000)` and never receives UI page, size, search, facet, favorite, or playlist constraints.

## Proposed Approach

Use server-backed numbered pagination. Extend the typed Music query/result and catalog API with bounded page metadata, count the complete criteria, then fetch one stable sorted page. Use the same catalog paging boundary for All Music and Favorites. Resolve a selected global playlist to its bounded track-ID set before catalog paging so playlist pages remain complete without exposing paths or downloading the whole catalog.

The browser owns only current query/view/page state. It requests one page at a time, validates all pagination metadata, renders the true result total, and shows a compact window of page buttons. History remains a separate bounded view. Radio remains server-owned and independent.

## Options Considered

- Server-backed numbered pages: selected because it provides totals, direct navigation, bounded responses, and predictable mobile behavior.
- Load all tracks and paginate in JavaScript: rejected because response and browser work grow with the entire library.
- Cursor-based infinite loading: rejected because it does not match the requested paging model and makes direct page navigation and total-page display less clear.

## Files or Modules Involved

- Music catalog query/result/service and read controller/view models.
- Music library playlist lookup boundary where playlist IDs are resolved.
- Music API URL builder, response validator, page state/rendering, template, and responsive styles.
- Java and JavaScript catalog/pagination/radio regression tests.
- Music package and frontend documentation.

## Validation Plan

- RED test proves a 1,549-track match reports the full total while returning only the requested 50-track page.
- Tests cover first, middle, final, empty, excessive, and negative page inputs; stable tie ordering; full-result facets; favorite and playlist paging; and search reset.
- JavaScript tests cover response validation, compact page windows, disabled boundary controls, current-page indication, mobile-safe markup, and preserved filter parameters.
- A radio regression verifies a paged catalog request cannot affect the separate 10,000-candidate query.
- Run focused Java and JavaScript tests, then the full website check and a non-8080 runtime request against the paged endpoint.
- Publish one focused PR, require platform CI, dependency review, and CodeQL, merge, allow automatic deployment, and verify the production total/pages plus radio playback.

## Acceptance Criteria

- All Music reports the full indexed match count rather than 98 or an unlabelled 100-result cap.
- Fifty or fewer tracks render per page, with correct navigation through every page.
- Search, facets, Favorites, and playlists return complete paged results.
- Navigating catalog pages does not interrupt active media.
- Radio remains eligible to select any of the approximately 1,500 present, ready, non-excluded tracks, regardless of the page visible to a listener.
- Authorization, inline-only playback, security headers, CI, and production smoke checks pass.

## Open Questions

None. The user approved numbered server paging and explicitly required radio to remain full-library.
