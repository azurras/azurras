# 2026-07-26 VIN Scheduling and Link Preview Issues 1176-1181

## 06:30 - Completed final issue batch

### Request

Complete every remaining open `azurras/christopherbell.dev` GitHub issue without routine approval
stops. The final batch covered issues #1176 through #1181. Preserve unrelated dirty state, use an
isolated worktree, carry the work through tests, PR/CI/merge, production-safe deployment, issue
closure, and Builder closeout.

### Project Context

- Builder hub: `C:\Users\Christopher\Developer\builder`, branch `main`.
- Spoke worktree: `A:\Projects\christopherbell.dev-worktrees\vin-schedulers-link-preview-1176-1181`.
- Spoke branch: `codex/vin-schedulers-link-preview-1176-1181`.
- The authoritative checkout at `A:\Projects\christopherbell.dev` contained extensive unrelated
  dirty and staged state. It was used only for read-only production tooling and was not altered.
- Production runs as native Windows services on this machine. Candidate verification used port
  `8092` and a disposable database before the live service on port `8080` was touched.
- Only `azurras` GitHub comments were eligible to direct work. PR #1257 had no comments, reviews,
  or review threads.

### Work Completed

- Issue #1176: made VIN decode cache entries decoder-version-aware with refreshed/expiry times,
  stale refresh behavior, and an expiry TTL index. Failed refreshes do not extend cache lifetime.
- Issue #1177: added ordered VIN batch DTOs and `POST
  /api/vehicles/2026-07-26/vin/decode/batch`; duplicates and invalid/null entries retain position,
  misses share one upstream batch call, errors are per input, and rate cost is weighted by input.
- Issue #1178: converted VIN collector settings to validated typed durations/limits, disabled
  RandomVIN by default, and established safe initial, fixed-delay, daily-cap, and lease defaults.
- Issue #1179: added shared renewable Mongo collector leases, durable run outcomes
  (`RUNNING`, `SUCCEEDED`, `FAILED`, `SKIPPED_LOCKED`), deterministic lease names, owner-scoped
  renewal/release, and protected Canes manual-collection contention behavior.
- Issue #1180: added link-preview destination policy and bounded HTTP client defenses for scheme,
  userinfo, localhost, private/link-local/multicast/reserved/documentation addresses, mixed DNS,
  redirects, DNS resolution, connect/read/overall timeout, content type/length, and streamed bytes.
- Issue #1181: added Mongo success/failure preview caching with explicit TTLs, safe failure
  categories, bounded URLs per post, and cache-outage degradation.
- Added migration V003 with checksum
  `799e5a12c1bfc022217a2c9f1e29f50ed4eef9b7f03daba01121a90c696dbd32` and three indexes:
  `vehicle_vin_cache_expiry`, `scheduled_collector_status_completed`, and
  `post_link_preview_cache_expiry`.
- Updated the affected VIN, scheduler, preview, and post documentation.
- Committed and pushed spoke commit `c1e9fc4f` (`Complete VIN scheduling and link preview
  hardening`).
- Opened PR [#1257](https://github.com/azurras/christopherbell.dev/pull/1257). All required CI
  gates passed, and it squash-merged as `9963ed0cc83f8b43f54612c1b8c6ed2966f22607`.
- GitHub automatically closed #1176, #1177, and #1178. Issues #1179, #1180, and #1181 remained
  open after merge and were queued for explicit evidence-backed closure.

### Decisions

- Kept the single-VIN route compatible and introduced batch decoding additively under the current
  dated API version.
- Preserved every submitted batch position, including duplicates and `null`, rather than making
  envelope validation erase useful partial results.
- Treated cache failures as per-input degradation while keeping envelope and rate-limit failures as
  request-level errors.
- Reused one shared lease/run-state coordinator across RandomVIN, NHTSA, Canes, and the existing
  restaurant collector contract instead of adding independent locking implementations.
- Validated the initial link-preview destination and every redirect target, bounded DNS lookup in
  the same overall deadline, and closed/cancelled slow body work promptly.
- Let the installed SYSTEM auto-deployer consume merged remote `main`; no production ACLs were
  weakened and no dirty checkout state was reset or overwritten.

### Validation

- Final `./gradlew.bat :website:check` passed in 1 minute 27 seconds.
- Java test XML: 147 suites, 1,200 tests, 0 failures, 0 errors, 3 skipped.
- JavaScript tests, `bootJar`, sensor-runtime verification, and `git diff --check` passed.
- Packaged candidate JAR started on `127.0.0.1:8092` as PID `55716` with profiles
  `local,deploy-smoke` and database `christopherbell_batch7_20260726`.
- Candidate batch API returned ordered `INVALID_VIN` entries for `"bad"` and `null`, rejected 21
  VINs with HTTP 400, preserved protected-state HTTP 403, and applied all V003 indexes.
- Candidate PID was stopped, port `8092` was freed, and MongoDB confirmed the disposable database
  was dropped. Production PID `41176` stayed healthy on port `8080` during candidate testing.
- PR CI passed Ubuntu, macOS, Windows, dependency review, CodeQL Java/Kotlin,
  JavaScript/TypeScript, Actions, and the aggregate CodeQL check.
- SYSTEM deployment rotated production from PID `41176` to PID `29164`.
- Production local and external roots and `/vin-decoder` returned HTTP 200. The production batch
  endpoint returned the same ordered two-error response and 21-entry HTTP 400. Protected vehicle
  state returned HTTP 403.
- MongoDB production migration V003 was `APPLIED` at `2026-07-26T11:27:46.286Z` with the exact
  checksum and expected index definitions.
- `ChristopherBellDev`, `MongoDB`, and `cloudflared` were all `Running` with `Automatic` start.
- Local runtime evidence is saved at
  `docs/test-reports/2026-07-26-vin-schedulers-link-preview-issues-1176-1181.md`.

### Current State

- PR #1257 is merged; production is running the merged behavior on PID `29164`/port `8080`.
- Builder plan:
  `docs/implementation-plans/2026-07-26-vin-scheduling-link-previews-issues-1176-1181.md`.
- Builder test report:
  `docs/test-reports/2026-07-26-vin-schedulers-link-preview-issues-1176-1181.md`.
- The isolated spoke worktree is clean and retained for provenance until campaign closeout.
- The authoritative spoke checkout remains dirty and untouched.

### Follow-ups

- Post evidence-backed closure comments and close #1179, #1180, and #1181.
- Update the final batch plan/ledger and save spoke review/update and hub closure artifacts.
- Verify the repository has zero open issues, refresh Builder indexes, validate Builder state, and
  commit/push the final campaign closeout.

## 06:34 - Closure addendum

- GitHub completed delayed automatic closure of #1179-#1181; full production evidence comments
  were posted to all three issues after closure.
- Live repository inventory returned no open issues. All 58 campaign issues are closed.
- The Batch 7 plan, campaign spec, and central work ledger were marked complete. Final spoke
  update/review and campaign work closure were saved for Builder closeout.
