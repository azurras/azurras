# christopherbell.dev What's for Lunch Legacy Location Reconciliation

## Document Status

complete

## Purpose

Finish the What's for Lunch location-integrity repair by retaining restaurants whose real locality can be proven, expanding configured OpenStreetMap coverage to the official places intersecting each configured import rectangle, correcting noncanonical legacy locations, and deleting only records that still have no authoritative place at their coordinates.

## Background

PR `#1342` removed the fabricated `Imported Metro, TX` fallback, added strict configured-locality validation, passed CI, merged as `178d90caca58d2f6284f54ab2ef4514d10df2918`, and was deployed. A backup-gated cleanup removed 6,825 exact `Imported Metro` rows, and the first strict production import completed successfully without recreating that exact placeholder.

Post-deployment inspection found a broader legacy population. A fresh read-only audit at 2026-08-02 16:47 America/Chicago classified all 7,268 OSM catalog rows against the current configuration and official U.S. Census Bureau geography:

- 5,493 rows already have a supported canonical city, state, country, and coordinates.
- 1,596 rows name a current 2025 Census incorporated place or Census-designated place inside a configured import rectangle, but are outside the current short city lists or contain a noncanonical state.
- 179 rows use a synthetic, misspelled, postal-community, airport, or otherwise unrecognized locality label.
- Point-in-polygon queries against the Census TIGERweb January 1, 2025 incorporated-place and Census-designated-place layers resolve 163 of those 179 rows to a real place.
- 16 rows have no Census place at their coordinates and remain unresolved.

The user explicitly selected **Expand and reconcile**: verify and add genuine nearby cities, correct reimported locations, and delete only synthetic or unresolved rows.

## Authoritative Geography Boundary

The reconciliation uses only current primary U.S. Census Bureau sources:

- 2025 Gazetteer place files for California, Louisiana, and Texas for canonical place names and identifiers.
- TIGERweb `Places_CouSub_ConCity_SubMCD` incorporated-place layer 4 and Census-designated-place layer 5, January 1, 2025 vintage, for rectangle intersection and point-in-polygon resolution.

The configured rectangles intersect 393 unique official places:

- Austin, TX: 70
- San Francisco Bay Area, CA: 154
- New Orleans, LA: 46
- Dallas, TX: 123

The source query returns two distinct California places named `Mountain View`; configuration stores the canonical name once because both resolve to the same city/state value. `Fairview`, `Rollingwood`, and `Sunnyvale` occur in more than one configured state, so locality resolution must use the restaurant coordinates and any supplied state rather than a global city-name overwrite.

## Goals

1. Treat every official incorporated place or Census-designated place intersecting a configured import rectangle as supported coverage for that metro.
2. Canonicalize city, two-letter state, and `US` country while accepting equivalent full state names from OSM.
3. Disambiguate same-name places using configured rectangle ownership and coordinates.
4. Reject a city tag whose coordinates fall outside the owning metro rectangle, even when the city name is configured.
5. Make the service persistence boundary independently enforce city, state, country, coordinate validity, and metro rectangle ownership.
6. Reimport after deployment so current OSM evidence corrects legacy rows where possible.
7. Use a reviewed, checksum-pinned Census resolution manifest to update remaining legacy OSM locations and delete only rows with no authoritative Census place.
8. Preserve dependent user/session data unless its exact restaurant record is deleted under the approved cleanup contract.

## Non-Goals

- Do not infer a city from a metro label, ZIP code, county, neighborhood, airport label, or nearest-place heuristic.
- Do not call Census or another geocoder from the normal application import path.
- Do not admit arbitrary OSM locality text merely because its coordinates are within a metro rectangle.
- Do not change public API response shapes, MongoDB document schemas, import scheduling, or the four configured rectangles.
- Do not delete any row that a fresh authoritative point-in-polygon lookup resolves to a supported place.
- Do not weaken production filesystem ACLs or bypass the protected Windows deployment workflow.

## Functional Requirements

### Coverage Configuration

1. `application.yml` and `WflProperties` defaults must contain the same 393 unique canonical place names grouped under the existing four metro rectangles.
2. Place names must come from the pinned Census coverage result, not from the observed MongoDB values alone.
3. Duplicate city names across states must remain valid configuration.
4. Duplicate city/state ownership across metros must remain invalid.

### Client Resolution

1. Read locality in the established order: `addr:city`, `addr:town`, `addr:village`, `addr:municipality`.
2. Validate coordinates before locality resolution.
3. Resolve locality candidates by normalized city name, then require coordinate containment in the candidate metro rectangle.
4. If OSM supplies a state, accept either the configured postal abbreviation or its full official name, and reject contradictions.
5. If OSM supplies a country, accept established United States aliases and reject contradictions.
6. Emit exactly one canonical city/state/`US` result; reject no-match or ambiguous-match candidates.

### Service Defense in Depth

1. A prepared import candidate is valid only when its canonical city and state belong to one configured metro and its coordinates fall inside that metro's rectangle.
2. Missing/invalid coordinates, unsupported city/state, wrong country, or out-of-rectangle coordinates increment `skippedInvalid` and are never persisted.

### Production Reconciliation

1. Take a new full production MongoDB backup immediately before reconciliation and prove a dry-run restore can read it.
2. Capture a fresh inventory and generate a deterministic manifest containing each target OSM ID, old location, Census source layer/GEOID when resolved, proposed canonical location, and action (`update` or `delete`).
3. Requery Census for every noncanonical target before mutation; abort if counts, source responses, or manifest checksum differ unexpectedly.
4. Deploy the expanded importer and complete a production preview/apply import before the direct reconciliation.
5. Update only exact manifest IDs with a resolved Census place; set canonical city/state/country and matching search fields without changing restaurant coordinates.
6. Delete only exact manifest IDs with no incorporated-place or Census-designated-place match. Delete their direct favorite and rating rows under the established cleanup contract; preserve historical session references.
7. Verify every remaining OSM row satisfies the deployed invariant and that no synthetic metro label remains.

## Proposed Approach

Represent supported client locations as a normalized city-to-candidate-list map. Each candidate owns canonical city/state plus its configured bounding box. Resolution filters by valid coordinates, optional state evidence, and rectangle containment. This removes the current last-write-wins behavior for same-name cities in different states.

Generate the four coverage lists once from pinned Census TIGERweb results and check them into both configuration surfaces. The application remains self-contained and does not depend on Census availability at runtime.

After the merged SHA is deployed, run a normal import so expanded current OSM tags update as many legacy documents as possible. Then rebuild the Census reconciliation manifest from the remaining noncanonical rows, take a fresh backup, perform exact-ID updates/deletes, and verify counts and public behavior.

## Files and Modules

- `website/src/main/resources/application.yml`
- `website/src/main/java/dev/christopherbell/whatsforlunch/restaurant/config/WflProperties.java`
- `website/src/main/java/dev/christopherbell/whatsforlunch/restaurant/OpenStreetMapRestaurantClient.java`
- `website/src/main/java/dev/christopherbell/whatsforlunch/restaurant/RestaurantService.java`
- `website/src/test/java/dev/christopherbell/whatsforlunch/restaurant/OpenStreetMapRestaurantClientTest.java`
- `website/src/test/java/dev/christopherbell/whatsforlunch/restaurant/RestaurantServiceTest.java`
- `website/src/test/java/dev/christopherbell/whatsforlunch/restaurant/WflPropertiesTest.java`
- `website/README.md`
- production-only ignored reconciliation evidence under the isolated worktree `build/` directory and `A:\Backups\christopherbell.dev`

## Validation Plan

1. Add failing client tests for newly covered places, full state names, duplicate city names across states, and out-of-rectangle city tags.
2. Add failing service tests for valid expanded coverage and out-of-rectangle prepared candidates.
3. Verify default and YAML coverage counts and exact city sets agree.
4. Run focused WFL tests, then `:website:test` and `:website:check` with a private Gradle user home.
5. Start the packaged app on a non-8080 port and isolated database with a deterministic Overpass fixture containing cross-state duplicate names, a new official place, a full state name, and invalid examples.
6. Save the local runtime test report, publish a PR, pass all required CI, merge, and deploy the exact SHA.
7. Verify production commit identity, readiness, liveness, public routes, Mongo health, and a bounded production preview/apply import.
8. Back up, generate/checksum/review the exact reconciliation manifest, apply it, and prove zero remaining invariant violations.
9. Exercise public nearby results in all four metros and confirm canonical locations.

## Acceptance Criteria

- The checked-in coverage contains exactly the 393 unique official places intersecting the four existing rectangles.
- Same-name cities in different states resolve correctly from their coordinates and do not overwrite one another.
- Full state names canonicalize to configured abbreviations; contradictory states remain rejected.
- Client and service both reject city/coordinate ownership mismatches.
- Automated, alternate-port runtime, CI, merge, and exact-SHA production evidence pass.
- A fresh full backup and restore dry run precede production reconciliation.
- Every retained OSM row has canonical city/state/`US`, valid coordinates, and coordinates inside the owning metro rectangle.
- Only fresh-manifest rows with no authoritative Census place are deleted.
- No `Imported Metro`, `Austin Metro`, wrong-state fallback, or other unresolved locality appears in the catalog or nearby results.

## Risks and Mitigations

- **Coverage drift:** Census geography changes annually. Pin the 2025 source evidence and record hashes; refresh deliberately in future work.
- **Same-name collision:** A global city map can silently choose the wrong state. Resolve a candidate list with coordinate ownership and test `Sunnyvale`, `Fairview`, and `Rollingwood`.
- **Rectangle edge behavior:** Official place polygons may cross a rectangle edge. Include places whose polygons intersect the fetch rectangle and require restaurant coordinates themselves to be inside the rectangle.
- **Irreversible cleanup:** Back up immediately before mutation, pin exact IDs and checksums, abort on drift, and retain restore instructions.
- **Reimport changes counts:** Treat the post-import inventory as the only valid input to the final manifest; do not reuse the discovery snapshot for mutation.

## Rollback and Recovery

- Code rollback: redeploy the prior known-good merged SHA if runtime verification fails.
- Data rollback: restore the new pre-reconciliation MongoDB backup if any exact-ID count or invariant check fails.
- Partial-operation recovery: make manifest operations idempotent and record matched/modified/deleted counts for each collection.

## Open Questions

None. The user approved expanded authoritative coverage and exact reconciliation.
