# 2026-08-09 - christopherbell.dev MongoDB Collection Catalog

## 16:20 - christopherbell.dev MongoDB Collection Catalog

### Request

Safely reduce the apparent MongoDB-container sprawl for the website, with explicit approval to proceed, safety enforcement, and targeted final fixes. The completed scope had to preserve live data and the dirty authoritative checkout while continuing through implementation, review, PR/CI, merge, protected production deployment, live metadata inventory, and Builder closure.

### Project Context

- Builder hub: `C:\Users\Christopher\Developer\builder`.
- Authoritative spoke: `A:\Projects\christopherbell.dev`, extensively dirty and intentionally untouched.
- Feature worktree: `A:\Projects\christopherbell.dev-worktrees\mongodb-collection-catalog`.
- Branch/base: `codex/mongodb-collection-catalog` from `2f025762e248cab5befe0fb699e0560f57006572`.
- Production was already one native MongoDB 8.3 service at `127.0.0.1:27017`, one `christopherbell` database, and one website service on 8080. The apparent sprawl was collections, not separate runtime containers.
- The feature worktree has an unavoidable unstaged `gradlew.bat` line-ending delta; it was never staged or committed.

### Work Completed

- Added `docs/operations/mongodb-collection-catalog.md`, a 51-row catalog with physical/logical names, owner/mapping provenance, role, retention, index, sensitivity, and status.
- Added `MongoCollectionCatalogTest` enforcing exact source/manual/shared mapping coverage and malformed-row/status rules.
- Added `VehicleProperties` startup validation that rejects equal vehicle import-state collection IDs.
- Added `prod.cmd mongo-inventory`, Make wiring, the fixed JavaScript metadata script, strict PowerShell canonicalizer, regular/view/capped/time-series support, nested redaction, deterministic indexes, BSON Long and safe-integer validation, exact `collStats` command enforcement, and two-host Pester coverage.
- Added repository and Windows operations documentation. No collection/document/schema/index mutation or application document read was introduced.
- Delivered ten feature commits from `5df1aa88` through `e5d90052`, PR [#1352](https://github.com/azurras/christopherbell.dev/pull/1352), and squash merge `0bcc8a9b83738df9c4adcf076e4be4443090448c`.
- Saved Builder test report, spoke update, branch review, closure, and updated spec/plan/work/task state.

### Decisions

- Preserve active physical collection boundaries. Consolidating collections would not reduce a MongoDB process/container because production already has one service/database; it would combine unrelated ownership, indexes, retention, and security contracts and add migration/rollback risk.
- Treat live-only or empty names as review inputs, never cleanup authority.
- Final-review fixes required explicit user authority after the planned final wave. The user authorized the residual task; final scoped re-review approved original-value floating-point validation and collation-strength enforcement.
- Protected ProgramData ACLs stayed intact. The public inventory wrapper failed closed for the non-elevated shell; the live check used the exact merged generator and canonical validator against the same fixed URI with the known `mongosh` path.

### Validation

- Fresh coordinator-owned `:website:check`: `BUILD SUCCESSFUL in 5m 7s`, 1,679 Java tests, zero failures/errors, and 83/83 production tests under both PowerShell hosts.
- Focused final operations: 69/69 in PowerShell 7 and Windows PowerShell 5.1.
- Disposable MongoDB 8.3.2 on 27018 verified regular/view/capped/time-series, BSON Long, sorting, redaction, and absence of document/view sentinels.
- Packaged candidate on 8097 returned liveness/readiness/home 200 and stable ZIP 404; candidate and disposable listeners stopped.
- PR and post-merge main Ubuntu/macOS/Windows, Dependency Review, and CodeQL gates passed.
- Protected deployment rotated PID 13484 to PID 62412. Mission Control and logs proved full SHA `0bcc8a9b83738df9c4adcf076e4be4443090448c`; local/public HTTP and Running/Automatic services passed.
- Live inventory generated at `2026-08-09T21:14:00.599Z`: 47 collections, 163 indexes, `actualOnly=[]`, zero sensitive scalar leaks, and four cataloged-but-uncreated names.

### Current State

- Website PR #1352 is merged and deployed; production liveness/readiness and public home are healthy.
- Builder `main` contains the completed test/update/review/closure records and is being validated/pushed as the final checkpoint.
- No source GitHub issue existed to close.
- The exact stopped disposable root `C:\Users\Christopher\AppData\Local\Temp\christopherbell-dev-mongo-catalog-final-runtime-e5d90052` remains at roughly 212 MB because recursive cleanup was policy-blocked.
- The external feature and deployment worktrees remain registered; the authoritative checkout remains untouched.

### Follow-Ups

- Use the catalog and metadata command for future ownership/drift audits.
- The four non-materialized names (`account_deletion_jobs`, `command_center_pending_actions`, `federation_scan_state`, `zip_coordinate_import_state`) are expected unexercised flows.
- Any future cleanup or consolidation requires separate explicit approval, a compressed SHA-verified and restore-tested backup, exact namespace/impact reporting, rollback retention, one-at-a-time action, and Mongo-backed production verification.
