# WFL Session and Restaurant Safety/Scalability Issues 1280-1289 Test Report

## Document Status

complete

## Story/Issue

GitHub issues `azurras/christopherbell.dev` #1280 through #1289.

## Branch

`codex/issues-1280-1289-20260729`, final source commit `f0312b8b0c60b9f25f633da46e421d4768d1ce69`, based on merged main `e3f7c676e8bf73a11056b9f009723ba9628025e8` and squash-merged as `b28031d535effef1fcbd547ba8f7dffdd4e76193`.

## App / Environment

The exact website JAR ran with profiles `local` and deployment-safe overrides at `http://127.0.0.1:8094` against disposable database `cbdev_issues_1280_1289_runtime`. Production port 8080 and its database were not changed during local acceptance. The final CodeQL follow-up removed ZIP storage from the client-only anonymous-session module; its focused test and the complete gate were rerun after that change, and production later served the exact normalized merged module.

## Local Run Details

The final acceptance process used PID `62200` on port 8094. Fixtures covered bounded restaurant inventory and duplicate groups, safe and unsafe URLs, active/expired/deletion-due sessions, twenty-member joins, revision conflicts, votes/resets, and account deletion. The exact process was stopped, port 8094 was confirmed free, and the 38-collection disposable database was dropped and confirmed empty.

## Test Cases

| Case | Expected | Result |
|---|---|---|
| Full gate | Java, JavaScript, packaging, sensor, and policy checks pass | Pass |
| Account deletion | Preserve creator-owned sessions and redact only the deleted participant | Pass |
| Atomic WFL mutation | Capped join, targeted vote, host-only reset, stable conflicts | Pass |
| Lifecycle | Active/archive/delete deadlines, audit, TTL, direct-read lag defense | Pass |
| Session list | One bounded hydration query group for 1 and 25 rows | Pass |
| Restaurant inventory | Stable bounded cursor pages and filters | Pass |
| Duplicate preview/apply | Mongo-grouped bounded pages and exact reviewed versions | Pass |
| Website URLs | Credential-free absolute HTTP(S) only at write/import/read/browser boundaries | Pass |
| Anonymous storage | Three IDs and 30-minute expiry only; no ZIP, coordinates, or full objects | Pass |
| Cleanup | Exact process, listener, and database removed | Pass |

## Data Sent

- Seeded four indexed restaurant records including safe and unsafe website values, two duplicate-name groups, and location search fields.
- Created active, expired, deletion-due, full, and mutable WFL sessions with explicit revisions and creator/member identities.
- Sent full and successful join calls, vote updates, host and non-host restaurant resets, and stale revision requests.
- Deleted one participating account and inspected the surviving shared-session document.
- Seeded legacy anonymous browser records with ZIP, coordinates, and full restaurant objects and exercised expiry, corruption, v2 canonicalization, and legacy migration.

## Response Received

The running app returned status code: 200 for root, inventory pages, duplicate preview, active session reads, successful join/vote/host reset, and account deletion. A full join returned status code: 409 with `WFL_SESSION_FULL`; a stale reset returned status code: 409 with `WFL_SESSION_CHANGED`; a non-host reset returned status code: 403; a deletion-due session returned status code: 404 while TTL lagged. Account deletion preserved the session, removed the deleted participant and vote, and advanced revision. Inventory returned stable two-row pages and location filtering; duplicate preview returned one bounded group. Unsafe restaurant websites were suppressed. Anonymous browser records canonicalized to v3 with only restaurant IDs and an expiry.

## Pass / Fail

All accepted cases passed. Final `:website:check` completed 1,489 Java tests with 0 failures/errors and 3 skipped; a direct complete browser run passed 288/288 tests. Boot JAR, sensor-runtime, and policy checks passed. `git diff --check` passed.

## Evidence

- Pull request [#1325](https://github.com/azurras/christopherbell.dev/pull/1325) passed Ubuntu, macOS, Windows, dependency review, and all CodeQL analyses with zero open alerts.
- Post-merge main CI and CodeQL passed for `b28031d535effef1fcbd547ba8f7dffdd4e76193`.
- Production listener rotated from PID 48420 to PID 52804; local/public root, WFL page, liveness, and readiness returned status code: 200.
- The public anonymous-session module exactly matched merged source after line-ending normalization, used v3, and contained no persisted ZIP field.
- Production migration 011 is `APPLIED` with checksum `73e242e0b87a60dea69ee9eaf7e3290c014891b5a306ac4d6c4df39c53fe2f2a`; lifecycle backfill gaps and unsafe websites each count zero; required WFL inventory/dedupe/participant/TTL indexes exist.
- Anonymous requests to the admin inventory and signed-in session list returned status code: 403.

## Bugs / Follow-ups

No unresolved Batch 4 defect remains. Continue with the eight still-open shared-folder issues #1290-#1297; #1298 was closed by the earlier security remediation.
