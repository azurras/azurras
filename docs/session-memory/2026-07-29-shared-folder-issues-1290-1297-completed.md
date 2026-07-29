# 2026-07-29 - Shared Folder Issues 1290 1297 Completed

## 14:39 - Shared Folder Issues 1290 1297 Completed

### Request
Continue the approved campaign to address every open `azurras/christopherbell.dev` website issue without stopping for routine approvals. This batch owned #1290-#1297 and required full implementation, local runtime evidence, PR/CI, merge/closure, production verification, and Builder continuity.

### Project Context
Work stayed in isolated worktree `A:\Projects\christopherbell.dev-worktrees\issues-1290-1297-20260729` on branch `codex/issues-1290-1297-20260729`, preserving the dirty authoritative checkout. Only issue bodies/comments from `azurras` were trusted; #1290-#1297 had no comments or attachments. Builder spec and reviewed implementation plan are `docs/specs/2026-07-29-shared-folder-integrity-retention-issues-1290-1297.md` and `docs/implementation-plans/2026-07-29-shared-folder-integrity-retention-issues-1290-1297.md`.

### Work Completed
- Added one asynchronous catalog worker with entry, directory, depth, scan-duration, cancellation, partial/freshness, and last-known-good semantics.
- Replaced capped search with deterministic path ordering, bounded page size, generation-bound opaque cursor, stable 409 on changed generations, and Load more browser UI.
- Added post-success catalog invalidation for create, rename, move, recycle, restore, purge, and durable upload completion.
- Wrapped download resources to emit exactly one COMPLETED, ABORTED, or FAILED audit terminal with actual bytes and without buffering.
- Added optimistic Mongo versioning and bounded conflict retries to shared-folder radio transitions; duration now resolves only from exact READY Music metadata and rejects forged reports outside tight tolerance.
- Added seven-day completed-upload TTL retention and cleanup-before-TTL terminal-media lifecycle with state-specific delays, artifact deletion, redaction, diagnostic retention, and maintenance ordering.
- Added migration 012 with bounded backfills, upload/media TTL indexes, media cleanup index, legacy radio-duration reset, and version initialization.
- Source commit `b93076780c9ff2925d98e650f4e4a6efc3148b24` was pushed, PR #1326 passed all gates, and squash-merged as `f67c90eed9b29215d562b2ac3670528f614508e9`. GitHub closed #1290-#1297.

### Decisions
Catalog scans never perform recursive filesystem work on request threads; a cold request receives BUILDING/empty and schedules work, while failures preserve the last immutable generation. Search cursors bind to generation, query hash, and stable path rather than offsets. Download source failures are distinct from client/early-close aborts. Terminal media records receive no TTL until artifact deletion succeeds; cleanup failures remain retryable and retain private metadata until safe deletion.

### Validation
- Final `:website:check`: 1,507 Java tests, 0 failures/errors, 3 skipped; JavaScript, boot JAR, sensor-runtime, and policy checks passed.
- Packaged PID 57180 on port 8095 used isolated database `cbell_issue_1290_1297_20260729` and disposable roots. Runtime proved BUILDING-to-FRESH catalog publication, stable two-page search, old-cursor 409 after mutation, generation advance, protected anonymous boundaries, four-byte resumable completion with seven-day TTL, and four-byte DOWNLOAD_COMPLETED audit.
- Candidate PID stopped, port 8095 freed, database dropped/absent, and generated runtime tree cleaned. Production remained healthy during testing.
- PR and post-merge Ubuntu/macOS/Windows builds, dependency review, and CodeQL passed.
- Production rotated from Java listener PID 52804 to 54448, serves merge-SHA assets locally/publicly, and reports root/shared/liveness/readiness 200. Migration 012 is APPLIED with expected checksum/indexes. Production invariant counts are zero for missing/unsafe upload and media TTL state. Website, MongoDB, and Cloudflared are Running/Automatic.
- Detailed evidence: `docs/test-reports/2026-07-29-shared-folder-integrity-retention-issues-1290-1297-test-report.md`.

### Current State
The batch worktree source is committed; the remote branch was deleted during merge. Builder test report was committed/pushed as `70650f5`. Production is healthy on the merged release. No batch test database, candidate listener, or runtime fixture remains.

### Follow-ups
Continue the approved website campaign with the remaining open issues, beginning at #1299. Refresh `origin/main`, use a new isolated worktree, preserve the authoritative checkout, and repeat the full delivery loop.
