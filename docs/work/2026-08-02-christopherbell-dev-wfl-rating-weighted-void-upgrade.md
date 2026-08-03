# christopherbell.dev What's for Lunch Rating-Weighted Void Upgrade

## Status

closed

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

- Project specification: [Rating-Weighted Void Upgrade](../specs/2026-08-02-christopherbell-dev-wfl-rating-weighted-void-upgrade.md), approved and ready for execution
- Implementation plan: [WFL Rating-Weighted Picks and Void Decision Console](../implementation-plans/2026-08-02-wfl-rating-weighted-void-upgrade.md)
- Test report: [WFL Rating-Weighted Void Upgrade](../test-reports/2026-08-02-wfl-rating-weighted-void-upgrade-test-report.md)
- Spoke update: [Completion update](../spoke-updates/2026-08-02-christopherbell-dev-wfl-rating-weighted-void-completion.md)
- Spoke review: [Final review](../spoke-reviews/2026-08-02-christopherbell-dev-wfl-rating-weighted-void-review.md)
- Closure: [Final closure](../work-closures/2026-08-02-christopherbell-dev-wfl-rating-weighted-void-closure.md)
- Session memory: [WFL rating-weighted Void upgrade](../session-memory/2026-08-02-wfl-rating-weighted-void-upgrade.md)

## Spoke Repositories

- `azurras/christopherbell.dev` at `A:\Projects\christopherbell.dev`

## Current State

PR [#1344](https://github.com/azurras/christopherbell.dev/pull/1344) passed the complete CI, dependency-review, and CodeQL suite, merged to `main` as `9c69623049829394f245515b8d1751c9f7579271`, auto-deployed, and passed public HTTPS and authenticated desktop/mobile verification. Production now uses the approved rating-weighted selection flows and scoped Void Decision Console.

## Blockers

None.

## Validation

Regression-first focused tests, all 313 JavaScript tests, the full `:website:check` gate, alternate-port runtime, independent code review, required GitHub checks, production health/API checks, and authenticated desktop/mobile browser checks passed. The reviewer-found authenticated rating-color cascade was corrected before publication and verified at 9.06:1 contrast.

## Next Steps

No required follow-up remains.
