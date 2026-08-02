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
- Scope amendment: [What's for Lunch Legacy Location Reconciliation](../specs/2026-08-02-christopherbell-dev-wfl-legacy-location-reconciliation.md)
- Implementation plan: [What's for Lunch Import Location Integrity](../implementation-plans/2026-08-02-christopherbell-dev-wfl-import-location-integrity.md)
- Reconciliation plan: [What's for Lunch Legacy Location Reconciliation](../implementation-plans/2026-08-02-christopherbell-dev-wfl-legacy-location-reconciliation.md)
- Test report: [What's for Lunch Import Location Integrity](../test-reports/2026-08-02-wfl-import-location-integrity-test-report.md)
- Reconciliation test report: [What's for Lunch Legacy Location Reconciliation](../test-reports/2026-08-02-wfl-legacy-location-reconciliation-test-report.md)
- Spoke task/update/review: pending implementation
- Closure: pending production cleanup and verification

## Spoke Repositories

- `azurras/christopherbell.dev` at `A:\Projects\christopherbell.dev`

## Current State

PR `#1342` removed the fabricated fallback, passed CI, merged as `178d90caca58d2f6284f54ab2ef4514d10df2918`, and deployed successfully. A backup-gated cleanup removed 6,825 exact `Imported Metro` rows, and a strict production import completed without recreating that placeholder.

A broader production audit then found 1,775 noncanonical OSM rows among 7,268. Current 2025 Census place files prove 1,596 already name real places. TIGERweb point-in-polygon queries resolve 163 of the remaining 179 to a real place, leaving only 16 without an incorporated-place or Census-designated-place match. The user approved expanding coverage and reconciling all resolvable rows instead of deleting the broader population.

## Blockers

None.

## Validation

Initial strict-import automated, alternate-port runtime, CI, merge, deployment, backup, exact-placeholder cleanup, and production import checks passed. Expanded-coverage implementation and alternate-port verification now also pass at spoke commit `1e7cd1daa066ff3ad386ed56f9391bd94c13bb03`; fresh PR/CI/deployment checks and final Census-backed reconciliation remain pending.

## Next Steps

1. Save and review the scope-amendment implementation plan.
2. Implement official rectangle-intersection coverage and coordinate-aware locality ownership regression-first in the existing isolated worktree.
3. Validate locally, publish, pass CI, merge, and deploy the exact SHA.
4. Reimport, take a fresh production backup, build a new exact-ID Census manifest, update resolvable rows, delete only unresolved rows, and verify the final invariant.
