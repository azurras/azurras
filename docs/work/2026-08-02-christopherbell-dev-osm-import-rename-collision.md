# christopherbell.dev OpenStreetMap Import Rename Collision

- Status: active
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
- Implementation plan: [OpenStreetMap Import Rename Collision](../implementation-plans/2026-08-02-christopherbell-dev-osm-import-rename-collision.md) (`ready-for-execution`).
- Test report: pending.
- Spoke update and review: pending.
- Closure record: pending.

## Current State

The root cause is confirmed. OpenStreetMap node `8178213204` changed from the persisted name `China Villa` to `Aama's Kitchen`, while node `13485126044` already owns normalized name `aama's kitchen` at a different location. `RestaurantService.applyPreparedImport` checks an incoming candidate by ID first and merges it without checking whether another ID owns the new normalized name. MongoDB correctly rejects the replacement through the unique `normalizedName` index, and the workflow records the entire startup catch-up run as failed.

The user approved preserving the unique-name invariant and skipping only the conflicting rename so the rest of the import can complete. The written specification is approved, and the literal-line implementation plan is ready for review and execution.

## Blockers

None.

## Validation

- Current upstream OpenStreetMap data returns two distinct `Aama's Kitchen` nodes in the configured Bay Area bounds: `8178213204` in Livermore and `13485126044` near Hayward.
- Read-only production MongoDB inspection confirmed `osm:node:8178213204` is persisted as `China Villa` and `osm:node:13485126044` owns normalized name `aama's kitchen`.
- Clean isolated-worktree baseline `./gradlew :website:test --no-daemon` completed successfully in 4m12s.

## Next Steps

1. Validate and review the literal-line implementation plan.
2. Implement the regression test first, then the smallest service and documentation changes.
3. Run focused and full automated verification plus alternate-port runtime validation.
4. Publish, pass CI, merge, verify production, and save Builder closeout artifacts.
