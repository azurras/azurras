# 2026-08-02 - OpenStreetMap import rename collision fix

## 13:14 - OpenStreetMap import rename collision fix

### Request

The user asked whether the recurring `RestaurantImportWorkflowService` startup-catch-up error could be fixed, chose the recommended behavior, and approved delivery. The supplied production log was a MongoDB `DuplicateKeyException` on unique `whatsforlunch.normalizedName` value `aama's kitchen`. Required constraints were to preserve the unique-name rule, preserve dirty authoritative checkouts, use isolated worktrees and alternate-port validation, avoid direct production data rewrites, retain causal errors for genuine failures, and complete PR/CI/merge/production delivery.

### Project Context

Builder coordinates `azurras/christopherbell.dev`. The authoritative checkout `A:\Projects\christopherbell.dev` was dirty and stale, so work used `A:\Projects\christopherbell.dev-worktrees\restaurant-import-duplicate-name-20260802`. The Windows host is also production; deployment runs through the protected SYSTEM auto-deployer and native services. Non-elevated ACL denial under `C:\ProgramData\christopherbell.dev` is expected and was not weakened.

### Root Cause and Decision

Production and upstream inspection established that OpenStreetMap node `8178213204` changed from persisted `China Villa` to `Aama's Kitchen`, while node `13485126044` already owned normalized name `aama's kitchen` at another location. The import's ID-first branch mutated/saved the ID match without checking the incoming normalized-name owner, so MongoDB correctly rejected it and the workflow marked the whole catch-up failed. The approved behavior preserves global name uniqueness, skips only that conflicting rename before mutation, continues later candidates, reports preview unchanged/apply skipped existing, logs the expected branch at DEBUG, and leaves unrelated failures causal and visible.

### Work Completed

- Saved and executed the approved Builder spec and literal-line implementation plan.
- Added `hasConflictingNormalizedNameOwner` to `RestaurantService`, called before preview classification and before apply merge/save.
- Added exact `China Villa` / `Aama's Kitchen` regression with a later continuation candidate.
- Updated the restaurant feature README.
- Spoke implementation commit: `3fdbafc0809a290f09504bb7fb9f8d201fc75e25`.
- Opened ready PR `https://github.com/azurras/christopherbell.dev/pull/1341`; required CI passed.
- Squash-merged as `0dd388fb096c924453bdbab8b66a3215d3e63452`.
- SYSTEM auto-deployer built and cut over the exact merged release; Mission Control reports application commit `0dd388fb`.

### Validation

- Regression-first RED: preview incorrectly reported one update before the implementation.
- Focused collision test GREEN; all 56 `RestaurantServiceTest` tests passed.
- `:website:test --no-daemon`: exit 0, 2m35s.
- `:website:check --no-daemon`: exit 0, 2m58s.
- Alternate-port runtime on 8096 with isolated MongoDB and loopback Overpass: status `SUCCEEDED`, fetched 2, imported 1, updated 0, skipped existing 1; existing records unchanged, later candidate imported, readiness/nearby HTTP 200, no duplicate-key/workflow error; task processes stopped and isolated database dropped.
- Independent code review: no Critical, Important, or Minor findings.
- GitHub CI: Ubuntu/macOS/Windows builds, Dependency Review, and all CodeQL checks passed.
- Production catch-up: `SUCCEEDED`, `lastCompletedMonth=2026-08`, fetched 20,000, imported 296, updated 442, skipped existing 19,262, skipped invalid 0.
- Production collision records retained their original names, normalized names, addresses, and May 21 `lastUpdatedOn` timestamps.
- Readiness, liveness, local homepage, public homepage, and production nearby endpoint returned HTTP 200; required security headers were present.
- MongoDB, ChristopherBellDev, ChristopherBellMediaWorker, and cloudflared remained Running/Automatic.
- Refreshed Mission Control logs ended with successful import completion; literal current-release searches for `DuplicateKeyException` and `OpenStreetMap import failed` returned no records.

### Durable Artifacts

- Work: `docs/work/2026-08-02-christopherbell-dev-osm-import-rename-collision.md`
- Spec: `docs/specs/2026-08-02-christopherbell-dev-osm-import-rename-collision.md`
- Plan: `docs/implementation-plans/2026-08-02-christopherbell-dev-osm-import-rename-collision.md`
- Test report: `docs/test-reports/2026-08-02-openstreetmap-import-rename-collision-test-report.md`
- Spoke update: `docs/spoke-updates/2026-08-02-christopherbell-dev-openstreetmap-import-rename-collision-completion.md`
- Spoke review: `docs/spoke-reviews/2026-08-02-christopherbell-dev-openstreetmap-import-rename-collision-review.md`

### Current State and Follow-ups

Production is healthy on merged commit `0dd388fb`. No required code, data, operational, or issue follow-up remains. The feature worktree intentionally remains available for PR provenance; its only uncommitted status is the known checkout-only `gradlew.bat` line-ending artifact with an empty `--ignore-space-at-eol` diff. The detached deployment evidence worktree is `A:\Projects\christopherbell.dev-worktrees\osm-import-merge-0dd388fb`. Do not clean unrelated historical worktrees.
