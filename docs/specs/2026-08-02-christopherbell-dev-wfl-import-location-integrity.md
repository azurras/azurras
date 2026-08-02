# christopherbell.dev What's for Lunch Import Location Integrity

## Document Status

ready-for-review

## Purpose

Ensure every OpenStreetMap restaurant included in What's for Lunch has a genuine, supported city and state. Remove the fabricated `Imported Metro, TX` fallback, reject unresolved locations, and safely remove the existing placeholder population from production.

## Background

Current `origin/main` maps OpenStreetMap elements in `OpenStreetMapRestaurantClient`. When an element lacks `addr:city`, the client stores `Imported Metro`; when it lacks `addr:state`, it stores `TX`; and when it lacks `addr:country`, it stores `US`.

That behavior is factually wrong. The configured import covers Austin, the San Francisco Bay Area, New Orleans, and Dallas. It can therefore label California and Louisiana restaurants as Texas locations, and it exposes a synthetic city that does not exist. The public nearby query selects coordinate-bearing records without requiring a supported city, so placeholder records can appear to users.

The user chose strict correctness over retention: use genuine location evidence when it is available, otherwise do not import the restaurant. The user also approved removal of the already-persisted `Imported Metro` OpenStreetMap records after a production backup and an exact impact report.

## Goals

- Never create or update a restaurant with the synthetic city `Imported Metro`.
- Resolve a genuine locality from supported OpenStreetMap address tags without adding a new external service.
- Include only restaurants whose resolved locality belongs to the configured city/state coverage.
- Require valid coordinates for imported restaurants.
- Reject conflicting country or state evidence instead of silently rewriting it.
- Keep import preview and apply classifications consistent.
- Remove existing placeholder OpenStreetMap records and their direct favorite/rating references from production safely.
- Prove that the cleanup population is not recreated by a subsequent import.

## Non-Goals

- Do not introduce per-record reverse geocoding or a dependency on Nominatim or another geocoder.
- Do not guess a city from a metro bounding box, ZIP code, street, or nearest configured city.
- Do not expand the configured metro or city coverage.
- Do not require a street address when the genuine locality, state ownership, country, and coordinates are valid.
- Do not delete valid manually managed restaurants.
- Do not rewrite historical lunch-session documents; unavailable restaurant IDs already degrade to the remaining resolvable restaurants.
- Do not weaken import timeouts, response-size bounds, lease behavior, unique-name constraints, or error reporting.

## Requirements

### Genuine locality resolution

1. Resolve locality from the first nonblank OpenStreetMap tag in this order: `addr:city`, `addr:town`, `addr:village`, `addr:municipality`.
2. Normalize the resolved locality only for comparison; preserve the canonical spelling from the matching configured city in stored data.
3. Match the locality against the unique configured `Metro.cities` ownership map.
4. If no supported configured city matches, omit the OpenStreetMap element from the client result.
5. Do not use `addr:place` as a locality because it can represent the address thoroughfare/place component rather than a municipality.

### State and country integrity

1. Store the state owned by the matched configured city.
2. A blank OpenStreetMap `addr:state` is acceptable because configured city ownership supplies the state deterministically.
3. When `addr:state` is present, accept it only when its normalized value equals the configured state abbreviation; otherwise omit the element.
4. A blank OpenStreetMap `addr:country` is acceptable because every configured metro is United States coverage.
5. When `addr:country` is present, accept case-insensitive `US`, `USA`, or `United States`; otherwise omit the element.
6. Remove the `Imported Metro`, `TX`, and generic default-text address fallbacks from the client.

### Coordinate and import classification

1. Require finite latitude in `[-90, 90]` and longitude in `[-180, 180]` before returning an imported restaurant.
2. Continue allowing a missing street or postal code when locality, state ownership, country, and coordinates are valid.
3. Elements omitted by the client are not fetched import candidates and therefore cannot be previewed, inserted, or used to update existing records.
4. Service-level import validation must independently reject any malformed imported restaurant that bypasses or does not originate from the OpenStreetMap client, including missing/unsupported city-state coverage or invalid coordinates.
5. Preview and apply must use the same service-level validity rule and count rejected candidates as `skippedInvalid` when they are present in a prepared snapshot.
6. Existing valid import matching, unique-name collision handling, counts, and continuation behavior remain unchanged.

### Existing production cleanup

1. Before mutation, capture a fresh production backup using the protected native-Windows operations workflow.
2. Query and report the exact count and representative samples for records meeting both conditions:
   - `_id` starts with `osm:`.
   - `address.city` equals `Imported Metro` exactly.
3. Record direct reference counts in `whatsforlunch_favorites` and `whatsforlunch_ratings` for the affected restaurant IDs.
4. Delete only the matched restaurant documents and their direct favorite/rating documents. Preserve all other restaurant and member data.
5. Preserve historical `whatsforlunch_sessions`; existing session rendering may omit deleted restaurants while retaining the session record and remaining choices.
6. Run cleanup only after the strict importer is deployed so the placeholder population cannot be recreated.
7. Record pre-cleanup counts, deleted counts, post-cleanup zero-count queries, backup evidence, and rollback instructions in the final test/closure evidence.

### Observability and documentation

1. Tests and import results must make unresolved-location rejection visible through counts, not fabricated output.
2. Genuine remote, parsing, persistence, lease, and workflow failures must retain causal diagnostics.
3. Update the restaurant package README to document supported locality tags, configured city ownership, coordinate requirements, and strict exclusion.
4. No public API response shape or MongoDB restaurant schema changes are required.

## Proposed Approach

Build a normalized, immutable city-coverage lookup from `WflProperties.Osm.metros` inside `OpenStreetMapRestaurantClient`. Each entry maps one configured locality to its canonical city spelling and configured state. Resolve an element's first genuine locality tag, look it up, validate optional state/country evidence, validate coordinates, and only then build the `Restaurant`.

Add a matching service-level predicate in `RestaurantService` so deterministic tests, future clients, or prepared imports cannot bypass the invariant. Keep client omission and service `skippedInvalid` classification separate: the client avoids manufacturing an invalid candidate, while the service remains the defense-in-depth persistence boundary.

Use a narrowly scoped, reviewable production cleanup operation after deployment. It will select exact OSM placeholder IDs, capture counts and samples, remove only those catalog documents and direct favorite/rating references, and verify zero remaining exact matches. The production backup is the rollback boundary.

## Alternatives Considered

### Recommended: strict source validation plus targeted cleanup

This removes fabricated data, uses only existing OSM and configured coverage evidence, adds no rate-limited dependency, and prevents recurrence. It intentionally drops incomplete or unsupported records.

### Reverse-geocode incomplete records

Coordinate-based geocoding could retain more restaurants, but up to 20,000 monthly candidates would add latency, rate-limit pressure, caching and attribution requirements, a new failure mode, and a second remote trust boundary. This is unnecessary for the approved correctness rule.

### Hide placeholder records at read time

Filtering nearby results would reduce immediate exposure, but corrupt records would remain in inventory, profiles, favorites, and future code paths. It treats a persistence defect as a presentation concern and does not meet the cleanup requirement.

## Files and Modules Involved

- `website/src/main/java/dev/christopherbell/whatsforlunch/restaurant/OpenStreetMapRestaurantClient.java`
- `website/src/test/java/dev/christopherbell/whatsforlunch/restaurant/OpenStreetMapRestaurantClientTest.java`
- `website/src/main/java/dev/christopherbell/whatsforlunch/restaurant/RestaurantService.java`
- `website/src/test/java/dev/christopherbell/whatsforlunch/restaurant/RestaurantServiceTest.java`
- `website/src/main/java/dev/christopherbell/whatsforlunch/restaurant/README.md`
- Existing native-Windows production backup/deployment tooling under `ops/production/windows/`
- Builder work, plan, test report, spoke update/review, session memory, and closure artifacts

## Validation Plan

1. Add a failing client regression proving missing locality no longer becomes `Imported Metro, TX`.
2. Add client tests for each supported locality tag, canonical configured spelling/state, unsupported locality, conflicting state, conflicting country, and invalid/missing coordinates.
3. Add a failing service regression proving an invalid prepared candidate increments `skippedInvalid` and is not saved.
4. Run the focused client and service test classes, full `:website:test`, and `:website:check` with a private Gradle home.
5. Start the packaged application with production-like configuration on a non-8080 port, isolated MongoDB database, and deterministic loopback Overpass response containing valid and invalid examples.
6. Exercise the import status/preview/apply path and a public nearby request; capture URL, port, request data, HTTP status/body, persistence counts, and logs.
7. Obtain independent code review, publish a pull request, wait for every required CI and CodeQL gate, and merge only after success.
8. Deploy the exact merged SHA through protected production operations and verify readiness, liveness, public routes, service state, MongoDB health, and application commit identity.
9. Take the production backup; capture exact placeholder and reference counts; execute the scoped cleanup; verify zero placeholder records and no unexpected count deltas.
10. Run or observe a bounded import after cleanup and prove no `Imported Metro` record is recreated and no placeholder appears in nearby results.

## Acceptance Criteria

- No application code contains an `Imported Metro` or default `TX` import fallback.
- Valid OSM records are stored with a canonical configured city and its actual configured state.
- Missing, unsupported, or contradictory location evidence is excluded rather than guessed.
- Invalid prepared candidates are counted as skipped-invalid and never persisted.
- Existing valid import behavior and API shapes remain compatible.
- Production is backed up before cleanup.
- The exact production OSM placeholder population and direct favorite/rating references are removed with recorded before/after counts.
- Historical sessions and unrelated data remain intact.
- A post-cleanup import does not recreate placeholder records.
- Focused, full, alternate-port runtime, independent review, CI, deployment, and production verification gates pass.

## Risks and Mitigations

- Risk: strict locality matching reduces catalog size. Mitigation: this is the user-approved correctness trade-off; alternate genuine OSM locality tags retain valid records without guessing.
- Risk: OSM supplies full state names instead of abbreviations. Mitigation: such candidates are conservatively excluded; a future separately designed normalizer may expand accepted verified forms.
- Risk: cleanup removes records referenced by users. Mitigation: count references first, delete direct favorite/rating rows with the same IDs, preserve historical sessions, and retain a pre-mutation backup.
- Risk: client and service validity rules drift. Mitigation: characterize both boundaries with shared acceptance cases and document the invariant in the package README.
- Risk: a partial or failed import is mistaken for successful cleanup validation. Mitigation: require durable successful import status plus database and HTTP evidence before closure.

## Rollback

- Code rollback: redeploy the previously verified production release if the strict importer causes a runtime regression.
- Data rollback: restore the pre-cleanup MongoDB backup if exact count reconciliation or dependent-data verification fails.
- Do not re-enable synthetic fallback values during rollback; pause imports instead if the previous release would recreate placeholders.

## Open Questions

None. The user approved strict genuine-location validation and existing placeholder cleanup on 2026-08-02.
