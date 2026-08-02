# 2026-08-02 - What's for Lunch location integrity and reconciliation

## 17:52 - What's for Lunch location integrity and reconciliation

### Request

The user reported that What's for Lunch invented `Imported Metro, TX` whenever source data lacked a restaurant location and required either a real location or exclusion. The user approved strict import enforcement, exact placeholder cleanup, and later selected the recommended expanded reconciliation: retain every restaurant whose real locality can be proven and omit only the rest.

### Project Context

Builder coordinated `azurras/christopherbell.dev`. The authoritative checkout `A:\Projects\christopherbell.dev` was dirty and preserved; implementation used isolated worktree `A:\Projects\christopherbell.dev-worktrees\wfl-import-location-integrity-20260802`. The Windows development host is also production, so candidate runtime testing used port 8098 and isolated MongoDB before protected deployment to port 8080. Production ACLs were not weakened.

### Work Completed

- PR #1342 removed the fabricated fallback, enforced supported canonical locations, added regression tests, merged as `178d90caca58d2f6284f54ab2ef4514d10df2918`, and deployed.
- A verified backup preceded deletion of 6,825 exact `Imported Metro` OSM rows; a strict reimport did not recreate them.
- Official Census 2025 Gazetteer and TIGERweb analysis expanded the four configured rectangles to 393 unique incorporated places and CDPs.
- PR #1343 added complete configuration coverage, coordinate-aware duplicate city resolution, full state-name aliases, city/rectangle contradiction rejection, independent service validation, tests, and README documentation. It merged and deployed as `1d1b322dc1667e48bc0230009a3fe79fce0a1b90`.
- The live import completed with fetched 10,796, imported 86, updated 136, skipped existing 10,574, skipped invalid 0.
- A new 2,050,494-byte backup was taken and passed `mongorestore --dryRun` before mutation.
- A fresh post-import TIGERweb manifest resolved 199 of 215 remaining violations and found no Census place for 16. Exact-ID drift checks passed; 199 were updated and 16 deleted. No favorites or ratings referenced the deleted IDs, and sessions were preserved.

### Decisions

- Never infer or fabricate a city from a metro label, ZIP code, neighborhood, airport name, or nearest-city heuristic.
- Use current official Census place polygons for correction and deletion decisions.
- Keep all place data pinned in the application; Census remains a build/reconciliation source rather than a runtime dependency.
- Disambiguate same-name places using coordinates and optional state evidence.
- Apply production changes only after a verified backup and an immutable checksum-pinned exact-ID manifest.

### Validation

- Focused final suite: 78 tests passed.
- Full `:website:test`: 1,620 tests, 0 failures, 0 errors, 3 skipped.
- Full `:website:check`: passed, including 76 Pester executions.
- Packaged candidate runtime on port 8098 passed with exact accepted/rejected fixture evidence; test processes stopped and isolated database dropped.
- Both PRs passed required CI and exact merged releases deployed healthy.
- Final production audit: 7,338 OSM rows, all 7,338 valid, 0 violations, 0 synthetic metro placeholders; 7,340 total restaurants.
- Readiness/liveness HTTP 200 `UP`, MongoDB ping `ok: 1`, listener PID `57904`.
- Public nearby requests for Austin, Bay Area, New Orleans, and Dallas returned HTTP 200 and canonical city/state/`US` results.

### Evidence and Recovery

- Backup: `A:\Backups\christopherbell.dev\christopherbell-before-wfl-legacy-reconciliation-20260802-224634.archive.gz`, SHA-256 `E8999314FC31EB440D5A142D317F628231B4B6BA25962C30FAA4F000CD92CD23`.
- Manifest: `A:\Backups\christopherbell.dev\wfl-production-reconciliation-manifest-20260802-224741.json`, SHA-256 `A6391EFC45FB88033B25DEA77A06C2C358E551A256EA69FFB59C53F624677918`.
- Receipt: `A:\Backups\christopherbell.dev\wfl-production-reconciliation-receipt-20260802-224800.json`, SHA-256 `F76FBD81E7401BBC85C3DB35EA52334E792D1B95DB541601D1E1197B44BA12E4`.
- Final audit: `A:\Backups\christopherbell.dev\wfl-production-final-invariant-audit-20260802-225100.json`, SHA-256 `E3E31D0B5F5C5A7283DA22DF1CFE5EB4EF234229CEFF9F2588F37B4857ACA055`.
- The backup is the rollback boundary if later data inspection discovers an unexpected issue.

### Current State and Follow-ups

Production is healthy on merged commit `1d1b322dc1667e48bc0230009a3fe79fce0a1b90`. The Builder work record is closed and no required follow-up remains. Refresh the pinned Census coverage only through a deliberate reviewed change when adopting a newer geography vintage. Preserve unrelated historical worktrees and the dirty authoritative checkout.
