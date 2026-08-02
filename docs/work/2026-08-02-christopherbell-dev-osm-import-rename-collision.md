# christopherbell.dev OpenStreetMap Import Rename Collision

- Status: ready-to-close
- Owner: Codex root agent
- Started: 2026-08-02

## Objective

Prevent an OpenStreetMap restaurant rename from aborting the monthly or startup catch-up import when another restaurant already owns the incoming normalized name, while preserving the existing unique-name rule and useful diagnostics for genuine failures.

## Scope

- Diagnose the supplied production `DuplicateKeyException` against current source, upstream OpenStreetMap data, and read-only production restaurant records.
- Apply the existing same-name/different-address skip rule to ID-based rename updates before any mutation or save.
- Keep preview and apply classifications consistent.
- Add focused regression coverage and update the restaurant feature documentation.
- Validate in an isolated worktree, run the app on a non-production port with isolated test data, deliver through pull request and required CI, merge, and verify production behavior.
- Do not relax the unique `normalizedName` index, merge restaurant identities, delete restaurant records, or rewrite production data.

## Spoke Repository

- Repository: `azurras/christopherbell.dev`
- Registered checkout: `A:\Projects\christopherbell.dev` (dirty and stale; preserve unchanged)
- Isolated worktree: `A:\Projects\christopherbell.dev-worktrees\restaurant-import-duplicate-name-20260802`
- Branch: `codex/restaurant-import-duplicate-name`
- Baseline: `origin/main` at `5bd14e994a6130a32166602a6f272581abc53525`

## Related Artifacts

- Project specification: [OpenStreetMap Import Rename Collision](../specs/2026-08-02-christopherbell-dev-osm-import-rename-collision.md) (`ready-for-execution`, approved 2026-08-02).
- Implementation plan: [OpenStreetMap Import Rename Collision](../implementation-plans/2026-08-02-christopherbell-dev-osm-import-rename-collision.md) (`in-progress`; Tasks 1 and 2 complete).
- Test report: [OpenStreetMap Import Rename Collision Test Report](../test-reports/2026-08-02-openstreetmap-import-rename-collision-test-report.md) (`complete`, PASS).
- Spoke update: [OpenStreetMap Import Rename Collision Completion](../spoke-updates/2026-08-02-christopherbell-dev-openstreetmap-import-rename-collision-completion.md) (`closed`).
- Spoke review: [OpenStreetMap Import Rename Collision Review](../spoke-reviews/2026-08-02-christopherbell-dev-openstreetmap-import-rename-collision-review.md) (no blockers or warnings).
- Session memory: [OpenStreetMap Import Rename Collision Fix](../session-memory/2026-08-02-openstreetmap-import-rename-collision-fix.md).
- Closure record: pending.

## Current State

The root cause is confirmed. OpenStreetMap node `8178213204` changed from the persisted name `China Villa` to `Aama's Kitchen`, while node `13485126044` already owns normalized name `aama's kitchen` at a different location. `RestaurantService.applyPreparedImport` checks an incoming candidate by ID first and merges it without checking whether another ID owns the new normalized name. MongoDB correctly rejects the replacement through the unique `normalizedName` index, and the workflow records the entire startup catch-up run as failed.

The user approved preserving the unique-name invariant and skipping only the conflicting rename so the rest of the import can complete. Implementation commit `3fdbafc0` added preview/apply collision guards, the exact regression, and feature documentation. PR [#1341](https://github.com/azurras/christopherbell.dev/pull/1341) passed all required checks and merged as `0dd388fb096c924453bdbab8b66a3215d3e63452`. The SYSTEM auto-deployer cut over that exact release, the production catch-up succeeded, both collision records remained unchanged, and fresh current-release log searches found no recurrence.

## Blockers

None.

## Validation

- Current upstream OpenStreetMap data returns two distinct `Aama's Kitchen` nodes in the configured Bay Area bounds: `8178213204` in Livermore and `13485126044` near Hayward.
- Read-only production MongoDB inspection confirmed `osm:node:8178213204` is persisted as `China Villa` and `osm:node:13485126044` owns normalized name `aama's kitchen`.
- Clean isolated-worktree baseline `./gradlew :website:test --no-daemon` completed successfully in 4m12s.
- Regression-first testing observed the expected RED preview mismatch, then the focused test and all 56 `RestaurantServiceTest` tests passed after the fix.
- Post-change `:website:test` passed in 2m35s and `:website:check` passed in 2m58s.
- Port 8096 readiness returned HTTP 200 `UP`; startup catch-up stored `SUCCEEDED` with fetched 2, imported 1, updated 0, and skipped existing 1; the public nearby endpoint returned HTTP 200.
- Cleanup released ports 8096 and 18996, stopped only recorded task processes, and dropped only `christopherbell_osm_collision_test_20260802`.
- GitHub Ubuntu, macOS, Windows, Dependency Review, and CodeQL checks all passed before merge.
- Production Mission Control reports application commit `0dd388fb`; catch-up status is `SUCCEEDED` with fetched 20,000, imported 296, updated 442, skipped existing 19,262, and skipped invalid 0.
- Production readiness, liveness, local/public homepage, and nearby API returned HTTP 200; required security headers were present; all four native services remained Running/Automatic.
- The two collision records retained their pre-deployment names, normalized names, addresses, and timestamps; current-release literal log searches returned no `DuplicateKeyException` or `OpenStreetMap import failed` records.

## Next Steps

1. Save the final hub closure record.
2. Mark the plan, specification, and central work record complete/closed.
3. Refresh indexes, validate hub state, and push the final Builder checkpoint.
