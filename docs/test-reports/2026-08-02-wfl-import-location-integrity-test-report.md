# WFL Import Location Integrity Test Report

## Document Status

complete

## Story/Issue

Builder work item `2026-08-02-christopherbell-dev-wfl-import-location-integrity`: stop OpenStreetMap imports from inventing `Imported Metro, TX`, accept only configured supported localities with valid coordinates, and store canonical city/state/country values.

## Branch

- Spoke branch: `codex/wfl-import-location-integrity`
- Commits under test: `28b4e5ff`, `35a3f34c`, and `42b77eed`
- Base: `origin/main` at `0dd388fb096c924453bdbab8b66a3215d3e63452`
- Spec: `docs/specs/2026-08-02-christopherbell-dev-wfl-import-location-integrity.md`
- Plan: `docs/implementation-plans/2026-08-02-christopherbell-dev-wfl-import-location-integrity.md`

## App / Environment

- App: `christopherbell.dev` Spring Boot website
- Profiles: `prod,deploy-smoke`
- Base URL: `http://127.0.0.1:8097`
- Loopback Overpass fixture: `http://127.0.0.1:18997/`
- MongoDB: `mongodb://127.0.0.1:27017`, isolated database `christopherbell_wfl_location_integrity_20260802`
- Explicit overrides: `SPRING_MONGODB_URI`, `SPRING_MONGODB_DATABASE`, `APP_MAIL_ENABLED=false`, `APP_FEDERATION_DISCOVERY_ENABLED=false`, `APP_MUSIC_ENABLED=false`, a temporary smoke-only JWT secret, monthly WFL import enabled, shared folder disabled, and Command Center disabled.
- Production listener `http://127.0.0.1:8080` remained on PID `35100` throughout the successful runtime test.

## Local Run Details

From `A:\Projects\christopherbell.dev-worktrees\wfl-import-location-integrity-20260802`, a hidden PowerShell runner invoked:

```powershell
.\gradlew.bat :website:bootRun --no-daemon "--args=--server.port=8097 --spring.profiles.active=prod,deploy-smoke --wfl.restaurant-import.monthly.enabled=true --wfl.restaurant-import.osm.endpoint=http://127.0.0.1:18997/ --logging.level.dev.christopherbell.whatsforlunch.restaurant=DEBUG --app.shared-folder.enabled=false --command-center.enabled=false"
```

The successful candidate Java PID was `18908`; the runner PID was `29876`. Logs and the captured Overpass request were stored temporarily under `build/codex-wfl-location-runtime/`. After verification, both candidate processes were stopped, ports `8097` and `18997` had no listener, and the isolated database was dropped with `{"ok":1,"dropped":"christopherbell_wfl_location_integrity_20260802"}` and `database_exists_after_drop=false`.

An earlier candidate attempt specified the isolated database only in the URI. The `prod` profile's separate `spring.mongodb.database` default selected `christopherbell`; that attempt only completed readiness/state reads, never called the fixture, and performed no import write. It was stopped before retrying with both Mongo settings explicitly overridden.

## Test Cases

1. Startup catch-up import accepts lowercase `addr:city=austin` and fallback `addr:town=oakland`, then stores canonical `Austin, TX, US` and `Oakland, CA, US`.
2. The same seven-element response excludes missing locality, unsupported Houston, contradictory Austin/CA, wrong country, and missing latitude records.
3. Alternate-port readiness returns healthy after the import.
4. Public nearby lookup returns the canonical imported restaurant around each test coordinate.
5. Candidate processes, fixture listener, and isolated database are removed without changing the production listener.

## Data Sent

The loopback fixture returned seven OpenStreetMap elements:

```json
{
  "elements": [
    {"id":9100000001,"lat":30.2672,"lon":-97.7431,"tags":{"name":"Canonical Austin Cafe","addr:city":"austin","addr:state":"TX","addr:country":"United States"}},
    {"id":9100000002,"lat":37.8044,"lon":-122.2712,"tags":{"name":"Canonical Oakland Cafe","addr:town":"oakland","addr:state":"CA","addr:country":"USA"}},
    {"id":9100000003,"lat":30.2673,"lon":-97.7432,"tags":{"name":"Missing Locality Cafe"}},
    {"id":9100000004,"lat":29.7604,"lon":-95.3698,"tags":{"name":"Unsupported Houston Cafe","addr:city":"Houston","addr:state":"TX","addr:country":"US"}},
    {"id":9100000005,"lat":30.2674,"lon":-97.7433,"tags":{"name":"Contradictory State Cafe","addr:city":"Austin","addr:state":"CA","addr:country":"US"}},
    {"id":9100000006,"lat":30.2675,"lon":-97.7434,"tags":{"name":"Wrong Country Cafe","addr:city":"Austin","addr:state":"TX","addr:country":"CA"}},
    {"id":9100000007,"lon":-97.7435,"tags":{"name":"Missing Coordinate Cafe","addr:city":"Austin","addr:state":"TX","addr:country":"US"}}
  ]
}
```

The fixture captured an Overpass form POST with a `data=` body of 1,869 bytes. Public requests had no authorization header or request body:

```text
GET http://127.0.0.1:8097/actuator/health/readiness
GET http://127.0.0.1:8097/api/whatsforlunch/restaurant/2026-05-17/nearby?latitude=30.2672&longitude=-97.7431&radiusMiles=15&useSavedPreferences=false
GET http://127.0.0.1:8097/api/whatsforlunch/restaurant/2026-05-17/nearby?latitude=37.8044&longitude=-122.2712&radiusMiles=15&useSavedPreferences=false
```

## Response Received

- Readiness: status `200`; body `{"status":"UP"}`.
- Austin nearby: status `200`; `success:true`; one payload entry named `Canonical Austin Cafe` with `city=Austin`, `state=TX`, `country=US`, latitude `30.2672`, longitude `-97.7431`.
- Oakland nearby: status `200`; `success:true`; one payload entry named `Canonical Oakland Cafe` with `city=Oakland`, `state=CA`, `country=US`, latitude `37.8044`, longitude `-122.2712`.
- Mongo `whatsforlunch` contained exactly two fixture documents: `osm:node:9100000001` and `osm:node:9100000002`, with those canonical locations.
- Mongo query for the five rejected names returned `[]`; exact placeholder query `{"address.city":"Imported Metro"}` returned count `0`.
- Import state was `SUCCEEDED`, trigger `startup-catch-up`, with `fetched=2`, `imported=2`, `updated=0`, `skippedExisting=0`, and `skippedInvalid=0`. Client-side exclusions occur before candidates reach the service counts.

Log excerpt:

```text
OpenStreetMap restaurant import fetched 2 candidates.
Saved OpenStreetMap restaurant id: osm:node:9100000001, name: Canonical Austin Cafe
Saved OpenStreetMap restaurant id: osm:node:9100000002, name: Canonical Oakland Cafe
OpenStreetMap restaurant import completed. Imported: 2, updated: 0, fetched: 2, skipped existing: 0, skipped invalid: 0.
```

## Pass / Fail

- Canonical locality import: **PASS** — supported lowercase city and alternate town tags mapped to configured city/state values.
- Invalid location exclusion: **PASS** — all five invalid elements were absent and no placeholder city was stored.
- Readiness: **PASS** — status 200 and `UP`.
- Public nearby API: **PASS** — both canonical imported restaurants were returned at their expected coordinates.
- Cleanup/isolation: **PASS** — candidate processes stopped, alternate ports released, isolated database dropped, and production 8080 remained on PID 35100.

Overall: **PASS**.

## Evidence

- Focused client/service tests: 68 passed.
- Full `:website:test --no-daemon`: exit 0 in 4m16s; 1,614 tests, 0 failures, 0 errors, 3 skipped.
- Full `:website:check --no-daemon`: exit 0 in 5m19s, including Java, JavaScript, deployment-build-context, sensor, and static-asset verification.
- `git diff --check origin/main...HEAD`: no errors.
- Runtime import began at `2026-08-02T16:07:31.529-05:00` and completed at `2026-08-02T16:07:31.547-05:00`.
- Cleanup verified only port 8080 listening afterward, owned by the unchanged production PID 35100.

## Bugs / Follow-ups

No remaining implementation defect was found in scope. Existing production placeholder records still require the separately approved backup-gated cleanup after the strict importer is deployed.
