# VIN Scheduling and Link Preview Issues 1176-1181 Test Report

## Document Status

complete

## Story/Issue

- [christopherbell.dev #1176](https://github.com/azurras/christopherbell.dev/issues/1176) - versioned VIN cache refresh and TTL lifecycle
- [christopherbell.dev #1177](https://github.com/azurras/christopherbell.dev/issues/1177) - ordered VIN batch decoding with per-input results
- [christopherbell.dev #1178](https://github.com/azurras/christopherbell.dev/issues/1178) - safe typed RandomVIN and NHTSA scheduler configuration
- [christopherbell.dev #1179](https://github.com/azurras/christopherbell.dev/issues/1179) - distributed collector leases and durable run state
- [christopherbell.dev #1180](https://github.com/azurras/christopherbell.dev/issues/1180) - link-preview SSRF defenses and bounded HTTP fetching
- [christopherbell.dev #1181](https://github.com/azurras/christopherbell.dev/issues/1181) - success/failure preview caching and fetch limits

## Branch

- Repository: `A:\Projects\christopherbell.dev-worktrees\vin-schedulers-link-preview-1176-1181`
- Branch: `codex/vin-schedulers-link-preview-1176-1181`
- Commit under test: `c1e9fc4f` (`Complete VIN scheduling and link preview hardening`)

## App / Environment

- App: packaged `christopherbell.dev` Spring Boot website
- Profiles: `local,deploy-smoke`
- Candidate URL: `http://127.0.0.1:8092`
- Candidate Java PID: `55716`
- Production URL preserved during testing: `http://127.0.0.1:8080`
- Production Java PID preserved during testing: `41176`
- Disposable MongoDB database: `christopherbell_batch7_20260726`
- NHTSA and RandomVIN scheduled collection disabled for the candidate runtime

## Local Run Details

The final `website/build/libs/website.jar` was started hidden from the isolated feature worktree
with the equivalent command:

```powershell
java -jar website/build/libs/website.jar `
  --server.port=8092 `
  --spring.profiles.active=local,deploy-smoke `
  --spring.mongodb.database=christopherbell_batch7_20260726 `
  --vehicles.nhtsa-vin.enabled=false `
  --vehicles.random-vin.enabled=false
```

The candidate started in 5.257 seconds and listened only on port `8092`. After verification, PID
`55716` was stopped, port `8092` was confirmed free, and the exact disposable database was dropped.
Production PID `41176` remained on port `8080`, and its root route continued to return HTTP `200`.

## Test Cases

| Case | Behavior | Result |
| --- | --- | --- |
| 1 | Complete Gradle verification, including Java and JavaScript tests | PASS |
| 2 | Public home renders from the packaged candidate | PASS |
| 3 | Public VIN decoder page renders and establishes the CSRF cookie contract | PASS |
| 4 | Existing single-VIN API rejects an invalid VIN | PASS |
| 5 | Additive batch API preserves input order and returns an error for both an invalid string and `null` | PASS |
| 6 | Batch API rejects an envelope containing 21 VINs before upstream work | PASS |
| 7 | Protected vehicle collection state remains inaccessible anonymously | PASS |
| 8 | V003 migration is recorded as `APPLIED` with the expected checksum | PASS |
| 9 | VIN and preview cache TTL indexes use `expireAfterSeconds: 0` | PASS |
| 10 | Collector-run status/completion compound index is present | PASS |
| 11 | Candidate teardown removes only the isolated process and database | PASS |
| 12 | Production listener and public root remain healthy throughout | PASS |

## Data Sent

Anonymous HTTP probes used the `XSRF-TOKEN` cookie from `GET /vin-decoder` and echoed it as
`X-XSRF-TOKEN` for POST requests, matching the browser client contract:

```text
GET /
GET /vin-decoder
POST /api/vehicles/2026-05-09/vin/decode
Content-Type: application/json
{"vin":"bad"}

POST /api/vehicles/2026-07-26/vin/decode/batch
Content-Type: application/json
{"vins":["bad",null]}

POST /api/vehicles/2026-07-26/vin/decode/batch
Content-Type: application/json
{"vins":["1HGCM82633A004352", ... 21 total entries]}

GET /api/vehicles/2026-05-09/data-collection-state
```

MongoDB verification read migration `003-ensure-vin-preview-collector-indexes` and enumerated
indexes on `vehicle_vin_decode_cache`, `scheduled_collector_runs`, and
`post_link_preview_cache` in the disposable database.

## Response Received

The running candidate returned HTTP status code 200 for both public pages and the following
complete status/body results:

```text
GET /                                                        -> 200
GET /vin-decoder                                             -> 200
POST /api/vehicles/2026-05-09/vin/decode                     -> 400
POST /api/vehicles/2026-07-26/vin/decode/batch (2 entries)   -> 200
POST /api/vehicles/2026-07-26/vin/decode/batch (21 entries)  -> 400
GET /api/vehicles/2026-05-09/data-collection-state           -> 403
GET / on production port 8080 after teardown                 -> 200
```

The two-entry batch response reported `submittedCount: 2`, `successCount: 0`, and
`errorCount: 2`. Result indexes `0` and `1` both had status `INVALID_VIN`; the first retained
submitted value `bad`, while the second retained a `null` submitted value. The oversized request
returned `INVALID_REQUEST` with `VIN decode batch cannot contain more than 20 VINs.`

Migration V003 reported:

```text
status: APPLIED
checksum: 799e5a12c1bfc022217a2c9f1e29f50ed4eef9b7f03daba01121a90c696dbd32
vehicle_vin_cache_expiry: { expiresOn: 1 }, expireAfterSeconds: 0
scheduled_collector_status_completed: { status: 1, completedOn: -1 }
post_link_preview_cache_expiry: { expiresOn: 1 }, expireAfterSeconds: 0
```

## Pass / Fail

PASS. The final packaged commit implements the VIN cache, batch, scheduler, collector-lease,
SSRF, and preview-cache contracts; passes the complete automated suite; starts cleanly against an
isolated database; creates the required migration indexes; preserves public and protected HTTP
boundaries; and leaves production traffic uninterrupted.

## Evidence

- `./gradlew.bat :website:check` exited `0` in 1 minute 27 seconds.
- Java test XML: 147 suites, 1,200 tests, 0 failures, 0 errors, 3 skipped.
- `:website:jsTest`, `:website:bootJar`, and `:website:verifySensorRuntime` completed successfully.
- `git diff --check` completed successfully.
- Runtime startup log recorded profiles `local,deploy-smoke`, port `8092`, PID `55716`, and a
  successful 5.257-second startup.
- MongoDB returned `{ ok: 1, dropped: 'christopherbell_batch7_20260726' }` for the exact disposable
  database after index inspection.
- Final listener enumeration showed port `8092` free and production port `8080` still owned by
  PID `41176`.

## Bugs / Follow-ups

- The application logs existing Hibernate Validator deprecation warnings for `@Valid` on list
  containers and the usual local SpringDoc exposure warnings. Neither warning is introduced by
  these issues or affects the tested behavior.
- Authenticated manual collector execution was not performed against the isolated runtime. Lease
  acquisition, renewal, owner-scoped release, contention, and durable run-state behavior are
  covered by the passing focused and complete Java suites.
