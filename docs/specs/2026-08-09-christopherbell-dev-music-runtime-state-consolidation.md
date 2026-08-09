# christopherbell.dev Music Runtime State Consolidation

## Document Status

`ready-for-review`

## Purpose

Reduce the website's MongoDB collection count by one while improving music-domain
organization and preserving current queue and radio behavior. The change consolidates
two compatible singleton-state namespaces into one physical collection without making
their updates share a concurrency token.

## Background

Production already uses one native MongoDB service and one `christopherbell` database;
the cleanup target is collection organization, not container or process consolidation.
A fresh metadata-only inventory on 2026-08-09 reported 47 live collections, 163 indexes,
and no unowned namespace. The inventory therefore found nothing that was safe to delete
as abandoned data.

The first compatible active pair is:

| Collection | Role | Live documents | Indexes | Current identity |
| --- | --- | ---: | ---: | --- |
| `music_queue_state` | singleton queue state | 1 | 1 | `_id: "global"` |
| `music_radio_state` | singleton radio state | 1 | 1 | `_id: "global"` |

The two collections have the same owner and lifecycle category, but they cannot simply
share their current collection name because their identical `_id` values would collide.
They also have separate optimistic-lock versions today; preserving that separation avoids
new contention between queue edits and radio transitions.

## Approved Decisions

- Optimize first for a simpler music-domain model.
- Consolidate only music runtime state in this first cleanup slice.
- Use one physical collection with two separate, validated envelope documents.
- Preserve independent queue and radio optimistic-lock versions.
- Retain the two old collections for seven days after production cutover.
- Make the old-collection drop a separate, explicitly approved destructive phase.

## Goals

- Introduce one canonical `music_runtime_state` collection.
- Preserve queue contents, radio state, and both current version values exactly.
- Preserve the behavior of queue operations, radio transitions, queue consumption,
  history idempotency, and cross-instance radio leasing.
- Keep queue and radio updates independently concurrent after consolidation.
- Provide a deterministic, fail-closed forward migration and a tested reverse conversion.
- After the approved observation and retirement phases, reduce production from 47 to 46
  live collections with no unowned namespace.
- Establish a small, evidence-backed pattern before considering any further merges.

## Non-Goals

- Do not merge music tracks, playlists, metadata edits, history, or access-attempt data.
- Do not consolidate collections from any other website domain in this slice.
- Do not change public music APIs, payloads, authorization, playback behavior, or UI.
- Do not combine queue and radio into one MongoDB document or one version counter.
- Do not drop, rename, truncate, or otherwise mutate either source collection during the
  initial cutover.
- Do not infer that approval of this specification authorizes the later destructive drop.

## Target Data Model

`music_runtime_state` contains exactly two application-owned documents:

```text
{
  _id: "queue",
  kind: "QUEUE",
  queue: { entries: [...] },
  version: <independent queue optimistic-lock version>
}

{
  _id: "radio",
  kind: "RADIO",
  radio: {
    stationSequence,
    trackId,
    observedToken,
    startedAt,
    durationSeconds,
    source,
    queueEntryId
  },
  version: <independent radio optimistic-lock version>
}
```

Each document must contain its expected `kind` and only its matching payload. A narrow
storage adapter performs exact-ID operations and rejects an ID, kind, or payload mismatch.
The broad cross-type repository operations that could accidentally enumerate or deserialize
the other document are not part of the music service interface.

The existing queue and radio domain models remain distinct. Their validation invariants
remain authoritative, including queue size and uniqueness limits and radio state/source
consistency. The shared collection is a storage boundary, not a combined aggregate.

## Forward Migration

The next immutable application migration performs the cutover while no old-version website
writer is active.

1. Read metadata and documents from the two literal source namespaces.
2. Require an expected source state:
   - each source has exactly one document;
   - each document has `_id: "global"`;
   - each document maps successfully through the current domain validator;
   - version values are present or absent only as allowed by existing Spring Data semantics.
3. Require `music_runtime_state` to be absent or to contain a fully equivalent, already
   completed two-document migration state. Reject a partial or conflicting destination.
4. Transform queue and radio into the two target envelopes, retaining all logical fields
   and version values.
5. Insert the target documents and then read them back through the production storage
   adapter.
6. Compare canonical logical payloads and versions with the sources.
7. Record the immutable migration as applied only after every check succeeds.
8. Leave both source documents and collections unchanged.

The migration must be safe to reevaluate after an interrupted startup. It may accept only
an absent destination or a fully verified equivalent destination; it must never guess how
to repair partial or divergent state.

## Runtime Storage Behavior

- Queue reads and saves address only `_id: "queue"` and `kind: "QUEUE"`.
- Radio reads and saves address only `_id: "radio"` and `kind: "RADIO"`.
- Each document retains its own optimistic-lock version and conflict behavior.
- Existing radio transition leasing and local locking remain unchanged.
- Existing radio-history writes remain in `music_radio_history` and are not part of this
  consolidation.
- A missing or malformed destination document fails clearly and does not fall back silently
  to stale source data.
- Runtime writes go only to `music_runtime_state` after cutover; the old collections are
  rollback snapshots, not dual-write targets.

## Catalog and Operational Visibility

- Add `music_runtime_state` as the active music runtime-state collection.
- Give the two source namespaces an explicit rollback-retained/retiring lifecycle in the
  collection catalog so inventory output distinguishes intentional retention from an orphan.
- Inventory remains metadata-only and must not emit document values.
- During the observation window, the expected physical count is 48: the new destination
  plus both retained sources.
- After retirement, the expected physical count is 46 and neither retired namespace may
  remain live.

## Rollback Design

A release rollback after destination writes requires a reverse conversion; simply starting
the old release would expose stale source state.

1. Stop all website writers.
2. Capture and checksum a fresh backup of `music_runtime_state` and both source namespaces.
3. Read and validate the current `queue` and `radio` destination documents.
4. Transform them back to the two source schemas with `_id: "global"`, preserving logical
   payloads and compatible version values.
5. Replace only the two exact source documents.
6. Read back and compare canonical payloads and versions.
7. Start the prior release only after verification succeeds.

The reverse conversion is implemented and tested before production cutover. A failed reverse
check leaves the writer stopped and reports a redacted operational error.

## Seven-Day Observation Window

For seven days after cutover:

- keep both old source collections unchanged;
- monitor readiness, liveness, music API behavior, optimistic-lock failures, migration
  status, radio transitions, queue operations, and unexpected MongoDB errors;
- periodically verify the destination still contains exactly the two expected identities;
- treat any evidence of lost queue/radio state or changed public behavior as a rollback
  trigger;
- do not claim the physical cleanup complete while the retained collections still exist.

## Retirement Phase and Destructive Boundary

Retirement is a separate operation after the observation window. Before requesting deletion
approval, provide an exact preview containing the production database, the two literal
collection names, current counts, current indexes, backup location, backup checksum, restore
test result, and expected post-drop collection count.

After explicit approval:

1. Stop website writers.
2. Take a fresh compressed backup containing `music_runtime_state`,
   `music_queue_state`, and `music_radio_state`.
3. Verify the backup checksum and restore it into a disposable database.
4. Prove restored logical payload equality and document counts.
5. Recheck that production contains the expected destination and source namespaces.
6. Drop only `music_queue_state`, then verify.
7. Drop only `music_radio_state`, then verify.
8. Start the website and verify runtime behavior, readiness, logs, inventory ownership, and
   the expected 46-collection total.

No wildcard, database-wide cleanup, or inferred namespace is allowed. If any precondition
changes after approval, stop and present a new preview rather than proceeding.

## Failure Handling

- Unexpected counts, IDs, shapes, types, versions, or destination state fail startup closed.
- Migration errors are redacted and recorded through the existing durable migration state.
- Source data is never deleted as error recovery.
- Production cutover does not proceed if clone migration or alternate-port acceptance fails.
- Retirement does not proceed if backup, checksum, restore, or readback evidence is missing.
- A transient readiness failure during listener rotation is rechecked; persistent failure
  triggers rollback.

## Expected Code and Documentation Areas

- `website/src/main/java/dev/christopherbell/music/radio/`
  - target runtime-state envelope, narrow storage adapter, and service wiring
- `website/src/main/java/dev/christopherbell/configuration/mongo/migration/`
  - immutable forward migration and reusable validated conversion logic
- `website/src/test/java/dev/christopherbell/music/radio/`
  - storage, behavior, concurrency, and rollback tests
- `website/src/test/java/dev/christopherbell/configuration/mongo/migration/`
  - migration precondition, fidelity, rerun, and partial-state tests
- `website/src/test/java/dev/christopherbell/architecture/`
  - collection catalog ownership and lifecycle coverage
- `docs/operations/mongodb-collection-catalog.md`
  - active and rollback-retained namespace ownership
- `docs/operations/mongodb-migrations.md` and the production runbook/scripts
  - cutover, reverse conversion, observation, retirement preview, backup, restore, and exact
    drop procedure

Literal file and line edit ranges belong in the implementation plan after the approved spec
is mapped against a fresh isolated worktree.

## Validation Plan

### Automated

- Forward conversion preserves every queue/radio logical field and version.
- Reverse conversion restores the old schemas and identities without loss.
- Migration succeeds from the verified source state.
- Migration accepts only a fully equivalent already-completed destination on reevaluation.
- Migration rejects wrong IDs, extra source documents, malformed payloads, conflicting
  destinations, and partial destinations.
- Queue and radio storage operations cannot address each other's documents.
- Queue optimistic-lock conflicts remain independent from radio conflicts.
- Existing queue, radio, history-idempotency, lease, API, architecture, catalog, migration,
  and operational-script tests remain green.
- Full website and repository verification pass on supported Windows and CI environments.

### Disposable MongoDB and Candidate Runtime

- Restore a current production backup into an isolated database.
- Run the candidate migration against the clone and verify exact source/destination counts,
  IDs, logical payload digests, versions, and indexes.
- Start the packaged website on a non-8080 port against that clone.
- Exercise queue reads/writes, radio reads/transitions, queued-track consumption, readiness,
  liveness, and relevant authenticated APIs with exact request/status/response evidence.
- Prove the reverse conversion against a separate clone before production approval.

### Production Cutover

- Record exact deployed commit, service PID rotation, port, migration record, collection
  metadata, destination identities, redacted logical digests, and endpoint results.
- Verify public and local health plus unchanged music behavior.
- Confirm required Windows services remain Running/Automatic and the current-release logs
  contain no migration, mapping, optimistic-lock, or MongoDB failures.

### Production Retirement

- Record the explicit deletion approval, exact backup path and checksum, disposable restore
  evidence, one-at-a-time drop results, final inventory, endpoint results, and service state.

## Acceptance Criteria

- The approved target collection has exactly the two expected, independently versioned
  documents.
- Queue and radio logical state match their pre-cutover sources at migration time.
- Public website and music behavior remain unchanged.
- The source collections remain intact for seven healthy days.
- No destructive operation occurs without the separate exact preview and approval.
- After retirement, production has 46 live collections, no unowned namespace, and a tested
  recoverable backup.

## Open Questions

None for design review. Implementation details and literal edit ranges will be resolved in
the implementation plan after this written specification is approved.
