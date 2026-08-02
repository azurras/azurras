# christopherbell.dev OpenStreetMap Import Rename Collision

## Document Status

Complete. The approved behavior was implemented, merged, deployed, and production-verified on 2026-08-02.

## Purpose

Make the OpenStreetMap restaurant import resilient when an already-persisted OpenStreetMap ID is renamed upstream to a normalized name owned by another restaurant. The candidate must be skipped consistently instead of violating MongoDB's unique-name constraint and aborting the complete scheduled import.

## Background

The supplied production log records a startup catch-up import failing in `RestaurantService.applyPreparedImport` with MongoDB error `E11000` for normalized name `aama's kitchen`.

The current data flow is:

1. Normalize the incoming OpenStreetMap restaurant name.
2. Look up a persisted restaurant by the incoming OpenStreetMap ID.
3. If that ID exists, merge all incoming import fields and save immediately.
4. Only when the ID does not exist, look up a restaurant by normalized name and apply the same-name/address duplicate rule.

Read-only production and upstream inspection confirmed the concrete collision:

- `osm:node:8178213204` is persisted as `China Villa` in Livermore, California, but OpenStreetMap now calls it `Aama's Kitchen`.
- `osm:node:13485126044` is a different physical location already persisted as `Aama's Kitchen` and owns normalized name `aama's kitchen`.
- The ID-first update path attempts to replace `China Villa` with the conflicting name and bypasses the duplicate-name rule used by the insert path.

This is a real import control-flow defect. Downgrading or suppressing the exception would hide a failed monthly import and leave its durable completion state incorrect.

## Goals

- Preserve the unique `normalizedName` invariant.
- Apply the same-name/different-address collision rule before an ID-based update mutates or saves a restaurant.
- Skip only the conflicting candidate and continue processing the remaining snapshot.
- Keep import preview counts consistent with apply results.
- Treat the expected collision as a concise non-error diagnostic while retaining `ERROR` and causal stack traces for genuine import failures.
- Prove the behavior with a regression test that models the exact upstream rename pattern.

## Non-Goals

- Do not change the unique MongoDB index or allow duplicate normalized names.
- Do not delete, merge, or re-key restaurant documents.
- Do not rewrite the existing `China Villa` or `Aama's Kitchen` production records.
- Do not change ratings, favorites, sessions, public API shapes, import scheduling, lease behavior, or OpenStreetMap query coverage.
- Do not catch and suppress arbitrary `DuplicateKeyException` instances; unexpected persistence failures must remain true errors.

## Requirements

### Matching and mutation

1. After normalizing a valid incoming restaurant, both preview and apply must resolve the current persisted record by ID and the owner of the incoming normalized name.
2. When the ID exists and a different persisted ID owns the incoming normalized name, the importer must classify the candidate as unchanged/skipped and must not mutate or save either persisted record.
3. When the ID exists and no different ID owns the name, the existing merge-and-save behavior must remain unchanged.
4. When the ID does not exist, the existing normalized-name and same-address behavior must remain unchanged.
5. The collision decision must occur before `mergeImportedRestaurant` to prevent an unsaved in-memory mutation from contaminating later logic or tests.

### Counts and logging

1. Preview must count an ID/name-owner collision as unchanged, not updated.
2. Apply must count the same collision as `skippedExisting`, not updated or imported.
3. Apply must emit a concise `DEBUG` diagnostic identifying the skipped incoming ID and normalized name without a throwable.
4. The workflow must continue processing subsequent candidates and, when no genuine failure occurs, record the import as completed.
5. Genuine remote, lease, validation, and persistence failures must retain their existing error handling and causal diagnostics.

### Compatibility

1. No repository method, public API, MongoDB schema, index, or durable state shape may change.
2. The implementation must stay within the existing restaurant feature ownership boundary.
3. The package README must document ID-based rename collision handling.

## Proposed Approach

Add a small private predicate in `RestaurantService` that receives the incoming normalized name and the matched restaurant ID, resolves the normalized-name owner through the existing `findRestaurantByNormalizedName` path, and returns true only when another ID owns that name.

Use the predicate in both classification phases:

- `prepareConfiguredMetroImport`: after an ID match, classify a different-name-owner collision as unchanged; otherwise preserve existing create/update/unchanged classification.
- `applyPreparedImport`: after an ID match and before merging, skip a different-name-owner collision, increment `skippedExisting`, log a throwable-free debug message, and continue.

This approach is intentionally narrow. It reuses the existing indexed lookup and duplicate policy, avoids schema or identity changes, and leaves MongoDB's unique constraint as defense in depth rather than normal control flow.

## Alternatives Considered

### Recommended: preserve unique names and skip the conflicting rename

This matches current product behavior, requires no migration, prevents the recurring error, and lets the remaining import complete. The trade-off is that the older persisted record can retain its prior name until an administrator deliberately reconciles it.

### Allow duplicate names by location

Changing uniqueness to a compound name/location identity would represent real-world franchises more naturally, but it would require index migration, revised repository methods, dedupe behavior, API assumptions, and broader production-data validation. The user did not select this scope.

### Reconcile or delete one identity automatically

Automatically merging or deleting documents could remove stale data, but it risks breaking ratings, favorites, sessions, and stable public restaurant IDs. Import-time destructive reconciliation is outside the approved behavior.

## Files and Modules

- `website/src/main/java/dev/christopherbell/whatsforlunch/restaurant/RestaurantService.java`
- `website/src/test/java/dev/christopherbell/whatsforlunch/restaurant/RestaurantServiceTest.java`
- `website/src/main/java/dev/christopherbell/whatsforlunch/restaurant/README.md`

## Validation Plan

1. Add a failing service test where an incoming ID matches persisted `China Villa`, the candidate name is `Aama's Kitchen`, and another ID owns normalized name `aama's kitchen` at a different address.
2. Assert preview reports unchanged rather than updated.
3. Assert apply reports one skipped-existing candidate, does not call `save`, and does not mutate either persisted record.
4. Include a following non-conflicting candidate and assert it is still saved, proving the collision does not abort the batch.
5. Run the focused `RestaurantServiceTest` class.
6. Run the full `:website:test` suite and repository checks required by the final diff.
7. Start the application with the production profile on a non-8080 port and an isolated MongoDB database, exercise the import workflow with deterministic collision data, and capture URL, request/input, HTTP status/body, application state, and logs.
8. After merge, deploy the exact merged SHA and verify local/public health, readiness, service state, successful catch-up completion, and absence of the duplicate-key signature during a bounded recurrence window.

## Acceptance Criteria

- The exact ID-based rename collision no longer calls `restaurantRepository.save` for the conflicting record.
- MongoDB emits no duplicate-key exception for the expected collision.
- Preview and apply agree that the candidate is unchanged/skipped.
- Later candidates in the same import are processed.
- The monthly/startup workflow can record successful completion.
- All focused, full, runtime, CI, and production verification gates pass.
- No schema migration or production restaurant mutation is required outside normal non-conflicting import updates.

## Risks and Mitigations

- Risk: preview and apply drift. Mitigation: use the same private collision predicate and assert both result counts.
- Risk: lookup cost increases for ID matches. Mitigation: use the existing unique indexed `normalizedName` query; the fallback exists only for legacy records missing the indexed field.
- Risk: stale restaurant display data remains. Mitigation: preserve non-destructive import behavior and leave explicit administrative reconciliation for separate scope.
- Risk: an unrelated persistence race still triggers a duplicate key. Mitigation: do not suppress arbitrary persistence failures; retain MongoDB uniqueness and existing error diagnostics.

## Open Questions

None. The user selected the recommended unique-name skip behavior on 2026-08-02.
