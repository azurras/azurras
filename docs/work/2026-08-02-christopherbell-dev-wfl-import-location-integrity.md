# christopherbell.dev What's for Lunch Import Location Integrity

## Status

active

## Objective

Eliminate fabricated `Imported Metro, TX` restaurant locations from What's for Lunch, prevent incomplete OpenStreetMap records from entering the catalog, and remove the existing placeholder records from production safely.

## Owner and Context

- Hub: `C:\Users\Christopher\Developer\builder`
- Spoke: `A:\Projects\christopherbell.dev`
- Requested by: user report on 2026-08-02
- Delivery model: isolated spoke worktree refreshed from `origin/main`, alternate-port application validation, pull request and required CI, merge, protected production deployment, and runtime verification

## Related Artifacts

- Spec: [What's for Lunch Import Location Integrity](../specs/2026-08-02-christopherbell-dev-wfl-import-location-integrity.md)
- Implementation plan: [What's for Lunch Import Location Integrity](../implementation-plans/2026-08-02-christopherbell-dev-wfl-import-location-integrity.md)
- Test report: pending implementation and local runtime testing
- Spoke task/update/review: pending implementation
- Closure: pending production cleanup and verification

## Spoke Repositories

- `azurras/christopherbell.dev` at `A:\Projects\christopherbell.dev`

## Current State

Root-cause inspection of current `origin/main` found that `OpenStreetMapRestaurantClient` explicitly substitutes `Imported Metro` when `addr:city` is missing and `TX` when `addr:state` is missing. The import covers Austin, the San Francisco Bay Area, New Orleans, and Dallas, so the fallback fabricates both locality and, for California and Louisiana candidates, state. Nearby selection uses coordinates without requiring a supported city, allowing these records to reach public results.

The user approved strict source validation, existing placeholder cleanup, and the written spec. The literal-line implementation plan passed mechanical validation and execution-readiness review with no blockers and is ready for execution.

## Blockers

None.

## Validation

Pending automated tests, alternate-port runtime verification with deterministic Overpass data, production backup and cleanup evidence, CI, merge, deployment, and public/local production checks.

## Next Steps

1. Commit and push the reviewed implementation-plan checkpoint.
2. Implement regression-first in an isolated spoke worktree.
3. Validate locally on a non-production port and save the test report.
4. Publish, pass CI, merge, deploy, back up production data, remove the exact placeholder population, and verify it is not recreated.
