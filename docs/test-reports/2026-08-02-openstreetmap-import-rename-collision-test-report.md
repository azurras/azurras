# OpenStreetMap Import Rename Collision Test Report

## Document Status

complete

## Story/Issue

Builder work item `2026-08-02-christopherbell-dev-osm-import-rename-collision`: prevent startup catch-up failure when an OpenStreetMap ID is renamed to a normalized name owned by another restaurant, while continuing later candidates.

## Branch

- Spoke branch: `codex/restaurant-import-duplicate-name`
- Commit: `3fdbafc0` (`Skip conflicting OSM restaurant renames`)
- Spec: `docs/specs/2026-08-02-christopherbell-dev-osm-import-rename-collision.md`
- Plan: `docs/implementation-plans/2026-08-02-christopherbell-dev-osm-import-rename-collision.md`

## App / Environment

- App: `christopherbell.dev` Spring Boot website
- Profiles: `prod,deploy-smoke`
- Base URL: `http://127.0.0.1:8096`
- Loopback Overpass stub: `http://127.0.0.1:18996/`
- MongoDB: `mongodb://127.0.0.1:27017`, isolated database `christopherbell_osm_collision_test_20260802`
- Overrides enabled monthly startup catch-up and RestaurantService DEBUG logging; mail, federation discovery, shared folder, Music, and Command Center were disabled.
- Production port 8080 and the production database were untouched.

## Local Run Details

From `A:\Projects\christopherbell.dev-worktrees\restaurant-import-duplicate-name-20260802`:

```powershell
.\gradlew.bat :website:bootRun --no-daemon "--args=--server.port=8096 --spring.profiles.active=prod,deploy-smoke --wfl.restaurant-import.monthly.enabled=true --wfl.restaurant-import.osm.endpoint=http://127.0.0.1:18996/ --logging.level.dev.christopherbell.whatsforlunch.restaurant.RestaurantService=DEBUG --app.shared-folder.enabled=false --command-center.enabled=false"
```

The candidate Java PID was `40916`; logs were captured under `build/codex-osm-collision-runtime/`. After testing, the exact app launcher, Java, and stub processes were stopped. Ports 8096 and 18996 had no listeners. The isolated database was dropped with `database_exists_after_drop=false`.

## Test Cases

1. Startup catch-up: seed `osm:node:8178213204` as `China Villa` / `china villa` and `osm:node:13485126044` as the owner of `Aama's Kitchen` / `aama's kitchen`; return an upstream rename of the first ID plus a later unique `Continuation Cafe`. Expect only the rename skipped, later import saved, state `SUCCEEDED`.
2. Readiness: expect alternate-port app and isolated MongoDB ready.
3. Public nearby lookup: expect preserved `China Villa` and imported `Continuation Cafe`.

## Data Sent

The loopback OpenStreetMap POST received a fixed JSON response containing:

```json
{"elements":[{"type":"node","id":8178213204,"lat":37.6807353,"lon":-121.7477737,"tags":{"amenity":"restaurant","name":"Aama's Kitchen","addr:housenumber":"4022","addr:street":"East Avenue","addr:city":"Livermore","addr:state":"CA","addr:country":"US","addr:postcode":"94550"}},{"type":"node","id":9999999001,"lat":37.682,"lon":-121.75,"tags":{"amenity":"cafe","name":"Continuation Cafe","addr:housenumber":"4100","addr:street":"East Avenue","addr:city":"Livermore","addr:state":"CA","addr:country":"US","addr:postcode":"94550"}}]}
```

Public requests, with no authorization header or body:

```text
GET http://127.0.0.1:8096/actuator/health/readiness
GET http://127.0.0.1:8096/api/whatsforlunch/restaurant/2026-05-17/nearby?latitude=37.6807353&longitude=-121.7477737&radiusMiles=15&useSavedPreferences=false
```

## Response Received

Readiness response status code: `200`; response body: `{"status":"UP"}`.

Nearby response status code: `200`; response body had `success:true` and payload entries for `China Villa` and `Continuation Cafe` at their expected Livermore addresses and coordinates.

MongoDB stored this completed state:

```json
{"_id":"openstreetmap-monthly","status":"SUCCEEDED","trigger":"startup-catch-up","lastCompletedMonth":"2026-08","lastResult":{"source":"openstreetmap","fetched":2,"imported":1,"updated":0,"skippedExisting":1,"skippedInvalid":0}}
```

Post-import rows proved `osm:node:8178213204` remained `China Villa` / `china villa`, `osm:node:13485126044` remained the Aama's owner, and `osm:node:9999999001` was saved as `Continuation Cafe` / `continuation cafe`.

Log excerpt:

```text
DEBUG ... Skipping OpenStreetMap restaurant id osm:node:8178213204 because normalized name aama's kitchen belongs to another restaurant.
INFO  ... Saved OpenStreetMap restaurant id: osm:node:9999999001, name: Continuation Cafe
INFO  ... OpenStreetMap restaurant import completed. Imported: 1, updated: 0, fetched: 2, skipped existing: 1, skipped invalid: 0.
```

No `DuplicateKeyException` or `OpenStreetMap import failed` line appeared.

## Pass / Fail

- Startup collision behavior: **PASS** — only the conflict was skipped, existing documents were preserved, the later candidate imported, and state was `SUCCEEDED`.
- Readiness: **PASS** — status code 200 and `UP`.
- Public nearby API: **PASS** — status code 200 with expected restaurants.
- Cleanup: **PASS** — processes stopped, ports released, isolated database dropped.

Overall: **PASS**.

## Evidence

- Regression-first test failed before the production fix: preview reported `updated=1` instead of expected `0`.
- Focused collision test passed after the fix; all 56 `RestaurantServiceTest` tests passed.
- `:website:test --no-daemon`: exit 0, `BUILD SUCCESSFUL` in 2m35s.
- `:website:check --no-daemon`: exit 0, `BUILD SUCCESSFUL` in 2m58s, including Java, JavaScript, deployment-context, sensor, and static-asset gates.
- `git show --check` and `git diff --check`: no committed whitespace errors.
- Runtime import began `2026-08-02T12:45:47.280-05:00` and completed `2026-08-02T12:45:47.300-05:00`.

## Bugs / Follow-ups

No remaining defect was found in scope. Subsequent delivery completed through PR #1341, merged commit `0dd388fb096c924453bdbab8b66a3215d3e63452`, and successful production catch-up verification.
