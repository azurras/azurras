# Shared-Folder Integrity and Retention Issues 1290-1297

## Document Status

ready-for-execution

## Purpose

Resolve GitHub issues #1290-#1297 by making the shared-folder catalog bounded and asynchronous, making search and radio stable across refreshes and instances, auditing actual transfer outcomes, and applying safe retention to completed uploads and failed media work.

## Context

The authoritative repository checkout is dirty and must remain untouched. Batch 5 uses an isolated worktree from merged `origin/main` commit `b28031d535effef1fcbd547ba8f7dffdd4e76193`. The eight issue bodies are authored by trusted user `azurras`; they have no untrusted comments or attachments. Issue #1298 was independently completed by PR #1324 and is outside this implementation batch.

The current catalog recursively walks the complete filesystem on a request thread and keeps a fifteen-second cache. Search returns at most 200 traversal-order matches without a cursor. Mutations do not invalidate the cache. Downloads audit only their start. Radio transitions use an in-process monitor and accept browser duration as authoritative. Completed uploads and non-ready terminal media jobs retain private metadata indefinitely.

## Goals

- Build catalog snapshots outside request threads with explicit entry, directory, depth, time, and cancellation budgets.
- Preserve and expose the last successful snapshot when refresh is partial or fails.
- Invalidate catalog generations immediately after committed create, rename, move, recycle, restore, purge, and upload-finalization operations.
- Page search deterministically with bounded sizes and an opaque cursor tied to one immutable generation.
- Reject a cursor after the generation changes rather than silently skipping or duplicating results.
- Record completed, aborted, and failed full/range transfers with actual bytes served and bounded deduplication.
- Make the single radio document optimistic-locking safe across instances with bounded conflict retries.
- Derive radio duration from revision-matching, server-indexed Music catalog metadata; browser reports are advisory and cannot control station timing.
- Retain completed upload sessions only for a short configurable window.
- Clean non-ready terminal media artifacts before retaining only redacted diagnostics and arming final TTL deletion.

## Non-Goals

- Persisting a second MongoDB copy of the complete shared-folder tree.
- Replacing the existing filesystem capability boundaries or the Music indexing pipeline.
- Making unknown or failed-to-index audio formats auto-advance from untrusted browser timing.
- TTL-deleting active, recoverable, or READY media cache records before existing cache eviction semantics permit it.
- Changing shared-folder read/write authorization policy.

## Design

### Asynchronous Catalog Generations

`SharedFolderCatalogService` owns one immutable snapshot and one bounded single-worker refresh. Requests never enumerate directories. A request returns the current or last-known-good snapshot and may trigger refresh. Invalidation increments the requested generation, cancels stale work, and schedules a replacement scan.

A scan stops when any configured maximum is reached: entries, directories, depth, elapsed time, or cancellation. Inaccessible directories mark the candidate partial and continue within budget; a root or systemic failure preserves the prior snapshot. Snapshot status exposes generation, creation time, freshness, partial state, and a fixed failure category without filesystem details.

An in-memory immutable snapshot is preferred over a Mongo catalog because the filesystem remains authoritative, snapshot size is explicitly bounded, and generation-bound cursors can provide stable paging without a second consistency protocol. On-demand bounded traversal is rejected because it still consumes request threads and cannot produce stable multi-page results.

### Search and Invalidation

Catalog entries sort by normalized relative path, exact path, type, and observation token. Search accepts a page size in a fixed range and an opaque Base64URL cursor containing the snapshot generation and last stable sort boundary. The server validates the cursor and returns 409 `SHARED_FOLDER_CATALOG_CHANGED` if its generation is no longer available. Authorization is refreshed on every page request.

The write and admin HTTP boundaries call the catalog invalidation interface only after a successful committed mutation. Upload completion invalidates only after the visible finalization is durable. Search UI preserves the cursor and offers a bounded Load More action; freshness/partial status is presented as plain text.

### Transfer Outcome Audit

A delegating Resource wraps the actual download stream. It counts bytes without buffering and records exactly one terminal outcome: completed when the advertised region is fully read, aborted when closed early, or failed on I/O/short-read failure. Completion callbacks use captured safe account/request facts, not thread-local request state. Existing logical-access windows deduplicate repeated browser media/range noise by account, action, and relative path.

### Radio Concurrency and Duration Trust

The radio document gains a Mongo optimistic version. Each `current` or `reportDuration` call reads, computes a pure candidate transition, and inserts/saves it with bounded retries on duplicate-key or optimistic-lock conflict. Exhaustion returns a stable service-unavailable response without overwriting another instance.

The trusted duration resolver maps `Music/...` catalog paths to `music_tracks` and accepts a duration only when the Music record is READY, present, and has the exact current observed token. A client report outside a tight tolerance is rejected; a matching report can trigger transition evaluation but the persisted duration remains the server value. Missing/unknown metadata leaves duration absent and the station playable without automatic forged advancement.

Optimistic locking is preferred over a distributed lease because the station is one low-contention document and every transition is deterministic from one observed version. A lease would add expiry/recovery state without improving the atomic compare-and-set boundary.

### Retention and Cleanup

Completed upload sessions receive a configurable `deleteAt` only after finalization is durable. Active, appending, finalizing, cancel-pending, and recoverable sessions never receive that TTL field. A migration backfills only completed legacy sessions and creates the TTL index.

FAILED, CANCELED, INSUFFICIENT_SPACE, and TIMED_OUT media jobs receive state-specific `cleanupAfter` deadlines. Bounded maintenance deletes descriptors, partials, cancellation/status files, and releases reserved/cache identity state idempotently. Only after artifact cleanup succeeds does it redact owner/source/private keys, retain status/failure category/timestamps, and set `deleteAt` for a short diagnostic window. Mongo TTL therefore cannot orphan artifacts. READY outputs remain governed by existing reader-aware cache eviction.

### Migration

Migration V012 creates upload/media deadline indexes and a radio version-compatible shape, then backfills bounded batches. It never assigns TTL to active upload/media work. Checksums are sealed by the existing migration framework and startup fails on partial or inconsistent migration state.

## Error Handling

- Catalog cold start returns a bounded empty BUILDING result; stale snapshots remain usable during refresh failure.
- Malformed/foreign cursors return 400; changed generations return stable 409.
- Catalog failure categories are fixed tokens and never expose absolute paths or exception text.
- Transfer audit persistence remains best effort and cannot break streaming.
- Radio conflicts retry a fixed number of times; unknown duration metadata does not accept client authority.
- Artifact cleanup failures retain the media document without `deleteAt` so later maintenance can retry safely.

## Testing

- Catalog: large trees, directory/entry/depth/time budgets, inaccessible directories, cancellation, failed refresh with last-known-good, immediate invalidation, deterministic duplicate-name paging, and generation change between pages.
- Web/UI: authorization on every cursor page, stable 400/409 envelopes, Load More behavior, stale/partial status, and request cancellation.
- Downloads: full, partial, exact byte counts, early close, short read, I/O failure, and logical range deduplication without buffering.
- Radio: two service instances racing `current` and duration calls, bounded conflict retry, forged short/long reports, exact trusted duration, stale observation token, and unknown format fallback.
- Retention: completed-upload TTL, active/recoverable immunity, state-specific media deadlines, crash before/after artifact cleanup, idempotent retry, redaction, reservation release, and TTL arming only after success.
- Migration: bounded backfill, exact indexes, active-state exclusions, rerun safety, and checksum contract.
- Final: focused tests, complete `:website:check`, exact JAR on a non-8080 port with a disposable database/tree, PR/CI/CodeQL, merge, issue closure, automatic production deployment, live authorization/freshness/retention/index checks, and exact cleanup.

## Acceptance

Every statement in #1290-#1297 has focused automated coverage and runtime evidence. No request thread performs recursive catalog enumeration; no browser controls radio duration; no TTL can remove active/recoverable work or orphan terminal artifacts; all required checks pass; the merged build is production-verified; and all eight issues contain closure evidence.
