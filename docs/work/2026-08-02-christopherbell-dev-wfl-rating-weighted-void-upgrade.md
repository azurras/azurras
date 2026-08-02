# christopherbell.dev What's for Lunch Rating-Weighted Void Upgrade

## Status

active

## Objective

Make every three-restaurant What's for Lunch draw rating-aware so highly rated restaurants appear more often and low-rated restaurants appear less often, while preserving randomness and discovery, and upgrade the page to the approved Void Decision Console visual direction.

## Owner and Context

- Hub: `C:\Users\Christopher\Developer\builder`
- Spoke: `A:\Projects\christopherbell.dev`
- Requested by: direct user request on 2026-08-02
- Delivery model: approved design, durable spec and implementation plan checkpoints, isolated spoke worktree from current `origin/main`, regression-first implementation, alternate-port browser verification, PR/required CI/merge, protected production deployment, and final runtime verification

## Approved Product Decisions

- Apply rating weighting to daily picks, nearby/ZIP picks, and deleted-pick replacements.
- Treat unrated restaurants as neutral rather than penalizing new places.
- Adjust sparse ratings toward neutral so one vote cannot dominate.
- Use moderate weighting: established 5-star restaurants approach twice neutral odds; established 1-star restaurants approach roughly one-third neutral odds.
- Select three unique restaurants through weighted sampling without replacement.
- Use the approved Void **Decision Console** direction while preserving all existing WFL controls and accessibility behavior.

## Related Artifacts

- Project specification: [Rating-Weighted Void Upgrade](../specs/2026-08-02-christopherbell-dev-wfl-rating-weighted-void-upgrade.md), pending user review
- Implementation plan: pending approved specification
- Test report: pending local application verification
- Spoke update/review and closure: pending implementation

## Spoke Repositories

- `azurras/christopherbell.dev` at `A:\Projects\christopherbell.dev`

## Current State

Current `origin/main` uses a uniform in-memory shuffle for both persisted daily picks and on-demand nearby/ZIP picks. Aggregate rating data already exists as per-restaurant count and sum. The current WFL template uses the light `site-page` shell and dedicated lunch CSS; the approved design moves it to the established dark Void visual system.

## Blockers

None. Implementation remains gated on the written-spec user review and the required implementation plan.

## Validation

Design exploration inspected current `origin/main`, the service selection paths, rating aggregation, WFL template/JavaScript/CSS, existing Void tokens, and recent WFL/Void commits. No source code has been modified.

## Next Steps

1. Save, self-review, commit, and push the approved project specification.
2. Obtain user review of the written specification.
3. Write, validate, review, commit, and push the implementation plan.
4. Implement and complete the full delivery loop.
