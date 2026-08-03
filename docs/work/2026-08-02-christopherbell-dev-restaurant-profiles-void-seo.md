# christopherbell.dev Restaurant Profiles Void and Search Indexing

## Status

active

## Objective

Render useful public restaurant profile content server-side so search engines can index every valid profile, add safe Restaurant structured data, and upgrade the profile presentation to the approved scoped Void design without exposing member-specific state.

## Owner and Context

- Hub: `C:\Users\Christopher\Developer\builder`
- Spoke: `A:\Projects\christopherbell.dev`
- Requested by: direct user request on 2026-08-02
- Delivery model: approved design, durable spec and implementation-plan checkpoints, isolated spoke worktree from current `origin/main`, regression-first implementation, alternate-port raw HTTP and browser verification, PR/required CI/merge, protected production deployment, and final runtime verification

## Approved Product Decisions

- Make every valid public restaurant profile indexable.
- Render the complete public profile in Thymeleaf and use JavaScript only for signed-in personal controls.
- Keep member rating/favorite state and audit fields out of the public page model.
- Emit safe conditional `Restaurant` JSON-LD while preserving canonical, sitemap, robots, and `404`/`noindex` behavior.
- Extend the WFL Void system to restaurant profiles with a single-restaurant layout.
- Leave Top Rated and Favorites outside the visual scope.

## Related Artifacts

- Project specification: [Restaurant Profiles Void and Search Indexing](../specs/2026-08-02-christopherbell-dev-restaurant-profiles-void-seo.md), ready for user review
- Implementation plan: pending written-spec approval
- Test report: pending implementation and local app testing
- Spoke update: pending implementation
- Spoke review: pending implementation
- Closure: pending final delivery
- Session memory: pending final delivery

## Spoke Repositories

- `azurras/christopherbell.dev` at `A:\Projects\christopherbell.dev`

## Current State

The three design sections have been approved in conversation. The consolidated written project specification is ready for user review. No spoke source changes or implementation worktree have been created for this initiative.

## Blockers

Implementation planning is intentionally waiting for the required user review and approval of the written project specification.

## Validation

Read-only exploration confirmed the current profile route, client-rendered template, progressive-enhancement opportunity, existing canonical/social-preview handling, public restaurant sitemap membership, robots behavior, and missing-profile `404`/`noindex` tests on current `origin/main`.

## Next Steps

1. Obtain user approval of the written project specification.
2. Refresh from `origin/main`, create an isolated spoke worktree, and inspect exact source/test line ranges.
3. Write, validate, and review the implementation plan before source edits.
4. Implement regression-first, run focused/full/alternate-port validation, publish and merge the PR, verify production, and close the Builder work record.
