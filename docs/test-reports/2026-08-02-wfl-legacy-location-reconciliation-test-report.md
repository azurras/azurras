# What's for Lunch Legacy Location Reconciliation Test Report

## Document Status

complete

## Story/Issue

Builder work item `2026-08-02-christopherbell-dev-wfl-import-location-integrity`, expanded by the approved legacy-location reconciliation spec: retain every OSM restaurant with an authoritative place, support all official places intersecting the configured rectangles, disambiguate same-name cities using coordinates, and reject city/coordinate contradictions.

## Branch

- Repository: `A:\Projects\christopherbell.dev-worktrees\wfl-import-location-integrity-20260802`
- Branch: `codex/wfl-legacy-location-reconciliation`
- Commit under test: `1e7cd1daa066ff3ad386ed56f9391bd94c13bb03`
- Base: deployed `origin/main` SHA `178d90caca58d2f6284f54ab2ef4514d10df2918`
- Packaged JAR: `website\build\libs\website.jar`
- JAR bytes: `128445991`
- JAR SHA-256: `E36FA576A17D9AC6A833D415BE9A1DC91A3BD5581F66109E478BB6D9F0EC6B80`

## App / Environment

- Application: `christopherbell.dev` Spring Boot website
- Candidate profiles: `prod,deploy-smoke`
- Candidate base URL: `http://127.0.0.1:8098`
- Candidate Java PID: `60304`
- Loopback Overpass fixture: `http://127.0.0.1:18998/`
- Isolated MongoDB: `mongodb://127.0.0.1:27017`, database `christopherbell_wfl_legacy_reconciliation_20260802`
- Production listener: `http://127.0.0.1:8080`, PID `13668`, unchanged throughout the successful test
- Relevant overrides: both `SPRING_MONGODB_URI` and `SPRING_MONGODB_DATABASE`, mail/federation discovery/music disabled, temporary smoke-only JWT secret, WFL monthly import enabled, shared folder disabled, and Command Center disabled

## Local Run Details

The packaged JAR was built with a private task-specific `GRADLE_USER_HOME`:

```powershell
.\gradlew.bat :website:bootJar --no-daemon
```

The loopback Overpass listener was started hidden from `build\codex-wfl-location-runtime\overpass-stub.ps1`. The app ran in the foreground-owned verification cell through `build\codex-wfl-location-runtime\app-runner.ps1`, whose effective Java command was:

```powershell
java -jar website\build\libs\website.jar `
  --server.port=8098 `
  --spring.profiles.active=prod,deploy-smoke `
  --wfl.restaurant-import.monthly.enabled=true `
  --wfl.restaurant-import.osm.endpoint=http://127.0.0.1:18998/ `
  --logging.level.dev.christopherbell.whatsforlunch.restaurant=DEBUG `
  --app.shared-folder.enabled=false `
  --command-center.enabled=false
```

Startup completed in 4.997 seconds and performed the bounded startup catch-up import. Logs and the captured Overpass request were stored temporarily under `build\codex-wfl-location-runtime\`.

After evidence collection, candidate PID `60304` was stopped, ports `8098` and `18998` had no listeners, the isolated database was dropped, and `database_exists_after_drop=false`. Production port `8080` remained on PID `13668`.

## Test Cases

1. Import lowercase `austin` with `TX` and canonicalize it to `Austin, TX, US`.
2. Import `Sunnyvale` without state at California coordinates and canonicalize it to `Sunnyvale, CA, US`.
3. Import the same `Sunnyvale` text without state at Texas coordinates and canonicalize it to `Sunnyvale, TX, US`.
4. Accept newly covered `Livermore` with full state name `California` and store `Livermore, CA, US`.
5. Accept newly covered `Fort Worth` with full state name `Texas` and store `Fort Worth, TX, US`.
6. Exclude missing locality, `Austin` at Dallas coordinates, contradictory Austin/CA, wrong country, and missing latitude.
7. Verify readiness and liveness.
8. Verify five public nearby requests return the expected canonical restaurant.
9. Verify exact Mongo rows, import state, invalid absence, and production isolation.

## Data Sent

The loopback fixture returned ten OSM elements:

```json
{
  "elements": [
    {"id":9200000001,"lat":30.2672,"lon":-97.7431,"tags":{"name":"Canonical Austin Cafe","addr:city":"austin","addr:state":"TX","addr:country":"United States"}},
    {"id":9200000002,"lat":37.3688,"lon":-122.0363,"tags":{"name":"California Sunnyvale Cafe","addr:city":"Sunnyvale"}},
    {"id":9200000003,"lat":32.7965,"lon":-96.5608,"tags":{"name":"Texas Sunnyvale Cafe","addr:city":"Sunnyvale"}},
    {"id":9200000004,"lat":37.6819,"lon":-121.7680,"tags":{"name":"Livermore Census Cafe","addr:city":"Livermore","addr:state":"California"}},
    {"id":9200000005,"lat":32.7555,"lon":-97.3308,"tags":{"name":"Fort Worth Census Cafe","addr:city":"Fort Worth","addr:state":"Texas"}},
    {"id":9200000006,"lat":30.2673,"lon":-97.7432,"tags":{"name":"Missing Locality Cafe"}},
    {"id":9200000007,"lat":32.7767,"lon":-96.7970,"tags":{"name":"Misplaced Austin Cafe","addr:city":"Austin"}},
    {"id":9200000008,"lat":30.2674,"lon":-97.7433,"tags":{"name":"Contradictory State Cafe","addr:city":"Austin","addr:state":"CA","addr:country":"US"}},
    {"id":9200000009,"lat":30.2675,"lon":-97.7434,"tags":{"name":"Wrong Country Cafe","addr:city":"Austin","addr:state":"TX","addr:country":"CA"}},
    {"id":9200000010,"lon":-97.7435,"tags":{"name":"Missing Coordinate Cafe","addr:city":"Austin","addr:state":"TX","addr:country":"US"}}
  ]
}
```

The fixture captured one 1,869-byte form POST whose decoded Overpass query contained all four unchanged configured rectangles.

Public requests had no authorization header or body:

```text
GET /actuator/health/readiness
GET /actuator/health/liveness
GET /api/whatsforlunch/restaurant/2026-05-17/nearby?latitude=30.2672&longitude=-97.7431&radiusMiles=15&useSavedPreferences=false
GET /api/whatsforlunch/restaurant/2026-05-17/nearby?latitude=37.3688&longitude=-122.0363&radiusMiles=15&useSavedPreferences=false
GET /api/whatsforlunch/restaurant/2026-05-17/nearby?latitude=32.7965&longitude=-96.5608&radiusMiles=15&useSavedPreferences=false
GET /api/whatsforlunch/restaurant/2026-05-17/nearby?latitude=37.6819&longitude=-121.7680&radiusMiles=15&useSavedPreferences=false
GET /api/whatsforlunch/restaurant/2026-05-17/nearby?latitude=32.7555&longitude=-97.3308&radiusMiles=15&useSavedPreferences=false
```

## Response Received

- Readiness: HTTP `200`, `{"status":"UP"}`.
- Liveness: HTTP `200`, `{"status":"UP"}`.
- Austin nearby: HTTP `200`, `Canonical Austin Cafe|Austin|TX|US`.
- California Sunnyvale nearby: HTTP `200`, `California Sunnyvale Cafe|Sunnyvale|CA|US`.
- Texas Sunnyvale nearby: HTTP `200`, `Texas Sunnyvale Cafe|Sunnyvale|TX|US`.
- Livermore nearby: HTTP `200`, `Livermore Census Cafe|Livermore|CA|US`.
- Fort Worth nearby: HTTP `200`, `Fort Worth Census Cafe|Fort Worth|TX|US`.
- Mongo contained exactly IDs `osm:node:9200000001` through `osm:node:9200000005` with the canonical values above.
- Mongo query for the five rejected fixture names returned count `0`.
- Import state: `SUCCEEDED`, trigger `startup-catch-up`, `fetched=5`, `imported=5`, `updated=0`, `skippedExisting=0`, `skippedInvalid=0`, completed `2026-08-02T22:24:00.618Z`.

Log excerpt:

```text
OpenStreetMap restaurant import fetched 5 candidates.
Saved OpenStreetMap restaurant id: osm:node:9200000002, name: California Sunnyvale Cafe
Saved OpenStreetMap restaurant id: osm:node:9200000001, name: Canonical Austin Cafe
Saved OpenStreetMap restaurant id: osm:node:9200000005, name: Fort Worth Census Cafe
Saved OpenStreetMap restaurant id: osm:node:9200000004, name: Livermore Census Cafe
Saved OpenStreetMap restaurant id: osm:node:9200000003, name: Texas Sunnyvale Cafe
OpenStreetMap restaurant import completed. Imported: 5, updated: 0, fetched: 5, skipped existing: 0, skipped invalid: 0.
```

## Pass / Fail

- Complete official coverage configuration: **PASS** — default and YAML coverage are recursively equal at counts `70, 154, 46, 123`, total `393`.
- Same-name cross-state resolution: **PASS** — California and Texas Sunnyvale both persisted with the correct canonical state based on coordinates.
- Expanded places/full state names: **PASS** — Livermore/California and Fort Worth/Texas persisted as CA/TX.
- City/rectangle contradiction: **PASS** — Austin at Dallas coordinates was absent.
- Other invalid exclusions: **PASS** — all five invalid fixture names were absent.
- Health/public API: **PASS** — both health endpoints and all five nearby requests returned HTTP 200 with expected bodies.
- Isolation/cleanup: **PASS** — candidate resources were removed and production PID `13668` remained the sole tested listener.

Overall: **PASS**.

## Evidence

- Initial red run: client failures proved same-name overwrite, out-of-rectangle acceptance, and expanded/full-state rejection; property test proved prior counts `15, 15, 8, 9`; service test proved missing rectangle enforcement.
- Focused final run: 78 tests passed after the new YAML binding test was added.
- Full `:website:test`: exit `0` in 2m45s; 1,620 tests, 0 failures, 0 errors, 3 skipped.
- Full `:website:check`: exit `0` in 3m42s; Java/JavaScript plus 76 Pester executions and deployment, sensor, and static-asset verification passed.
- `:website:bootJar`: exit `0`; packaged JAR hash recorded above.
- Official coverage comparison: exact matches for Austin 70, Bay Area 154, New Orleans 46, Dallas 123.
- `git diff --check`: no errors before commit.
- Runtime import began `2026-08-02T22:24:00.480Z` and completed `2026-08-02T22:24:00.618Z`.
- Cleanup: isolated DB dropped, `database_exists_after_drop=false`, only production `:::8080 PID=13668` remained among tested ports.

## Bugs / Follow-ups

- The first background wrapper launch was terminated before readiness and left no Spring failure; the same exact packaged JAR passed when kept foreground-owned by the verification cell. This was a harness/process-lifetime issue, not an application failure.
- Production reimport, new backup, exact Census manifest regeneration, reconciliation, and final zero-violation audit remain required after PR merge and deployment.
