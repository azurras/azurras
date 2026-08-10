# christopherbell.dev Domain Collection Consolidation

## Document Status

Ready for review.

## Purpose

Reduce the production `christopherbell` MongoDB database from 48 physical
collections to approximately 14 domain-owned collections, remove every
superseded source collection during the same guarded maintenance cutover, and
preserve the website's HTTP behavior, data, uniqueness constraints, retention
rules, and operational safety.

## Background

Production already uses one MongoDB process and one database. The first
consolidation moved one queue document and one radio document into
`music_runtime_state`, but retained the two legacy collections for rollback.
The user rejected that narrow result and approved an aggressive structural
consolidation with these decisions:

- Target approximately 15 physical collections; the proposed model has 14.
- Use one guarded maintenance cutover instead of a multi-release dual-write
  period.
- Delete superseded collections immediately after full migrated-state and
  runtime verification.
- Prefer a materially smaller, domain-oriented collection list over preserving
  one physical collection per Java document type.

The verified production baseline on 2026-08-10 is 48 collections and 164
indexes. The largest active datasets include 33,791 ZIP-coordinate documents,
7,340 restaurant documents, 7,035 shared-folder audit events, 4,328 music radio
history events, 2,284 scheduled collector runs, and 1,549 music tracks.

## Goals

1. Produce a stable physical collection model with approximately one collection
   per bounded domain.
2. Preserve every existing document and required index semantic.
3. Make document kind and ownership explicit in every shared collection.
4. Prevent cross-kind identifier collisions and cross-kind query leakage.
5. Remove all superseded source collections in the successful maintenance
   transaction window.
6. Leave production, automatic deployment, backup, rollback, and collection
   inventory tooling compatible with the new schema.
7. Present the MongoDB collection inventory as a concise domain-level view.

## Non-Goals

- Do not combine all data into one universal collection.
- Do not merge data across the `christopherbell` database boundary.
- Do not change public HTTP routes, request/response contracts, authorization,
  or user-visible feature behavior.
- Do not weaken unique constraints, TTL behavior, audit retention, or
  optimistic concurrency.
- Do not delete backup archives or unrelated historical release artifacts.
- Do not retain compatibility reads from superseded collections after the
  successful cutover.

## Target Physical Collections

| Target | Source document kinds and collections |
|---|---|
| `accounts` | accounts, account follows, trust relationships, account deletion jobs |
| `sessions` | browser sessions and conversation archive states |
| `communications` | messages, notifications, notification preferences, delivery guards, and rate limits |
| `content` | posts, post likes, post reports, hidden threads, and link-preview cache entries |
| `federation` | federation scan state and outbound delivery jobs |
| `music` | tracks, playlists, metadata edits, runtime state, radio history, and access attempts |
| `whatsforlunch` | restaurants, votes, favorites, preferences, sessions, daily picks, import state, and import previews |
| `shared_folder` | audit events, maintenance leases, media jobs, mutation recovery, radio state, recycle items, and upload sessions |
| `vehicles` | vehicles, VIN decode cache, and import state |
| `location` | ZIP coordinates and ZIP import state |
| `canes_box_tracker` | price snapshots |
| `application_runtime` | application leases and scheduled collector runs |
| `application_migrations` | migration ledger |
| `admin_activity` | administrative audit activity |

The successful cutover drops all source collections not present in this target
list, including `music_queue_state`, `music_radio_state`, and the intermediate
`music_runtime_state` collection.

## Shared Collection Contract

Every document in a shared physical collection must contain:

- `_id`: the canonical BSON document `{ kind: <_kind>, legacyId: <original
  BSON _id> }`, with fields encoded in that exact order, so identifiers cannot
  collide across document kinds and original BSON identifier types remain
  lossless;
- `_kind`: an exact, lower-case, allowlisted discriminator owned by the target
  domain;
- `schemaVersion`: the kind-specific persisted schema version;
- the original domain fields without lossy conversion.

Repository and store boundaries must always constrain reads, updates, deletes,
counts, and uniqueness checks by `_kind`. A generic unscoped repository API is
not allowed. Each domain owns one mapping layer that translates between domain
IDs and canonical persisted IDs. Runtime code must reject a persisted `_id`
whose `kind` member differs from the document's `_kind`.

Unknown kinds, malformed namespaced IDs, duplicate source IDs, unsupported
numeric BSON types, and unexpected source fields fail migration before any live
rename or deletion.

## Index Model

Each target collection has one `_id` index plus the union of required
kind-scoped indexes. Indexes that apply to only one kind must use an exact
`partialFilterExpression` on `_kind`. Unique indexes must remain unique within
their kind and retain their existing sparse or partial behavior.

The migration must build and validate all target indexes before live rename.
The index verifier compares keys, order, uniqueness, sparsity, partial filter,
TTL, collation, and index count. Index consolidation is allowed only when two
kinds have identical semantics; otherwise their partial indexes remain
separate.

## Migration and Maintenance Cutover

1. Acquire the protected deployment lock and suspend website service recovery.
2. Create and checksum a fresh `mongodump`; verify it with a dry-run restore.
3. Stop the website writer and prove production port 8080 is closed. MongoDB and
   cloudflared remain running.
4. Restore the backup to an isolated candidate database.
5. Run the complete consolidation against the candidate database using the
   exact release candidate.
6. Verify per-kind source/target counts, canonical BSON checksums, representative
   readbacks, indexes, and absence of unexpected collections.
7. Start the candidate application on the alternate port against the migrated
   candidate database and run the full smoke suite.
8. Re-run the migration against the stopped production database, writing only
   to uniquely named temporary target collections.
9. Re-run exact counts, checksums, document-shape validation, and index
   validation before publication.
10. Rename existing target-name collections to bounded temporary legacy names,
    then rename the validated temporary collections to the 14 target names.
11. Start the new release under an exact schema-direction marker and run local
    liveness, readiness, route, authentication-failure, and public endpoint
    checks.
12. Verify live per-kind counts/checksums and the exact 14-collection catalog.
13. Drop every bounded temporary legacy collection immediately.
14. Verify that no superseded collection exists, restore normal service
    recovery, refresh automatic-deploy tooling, and release the deployment lock.

No collection is dropped before the new release passes both candidate-database
and live-database verification.

MongoDB does not provide one transaction spanning all collection renames.
Publication therefore uses a durable, compare-and-set migration ledger with an
ordered manifest of all old, temporary, and final names. Each per-collection
rename is atomic, and the ledger records the next permitted operation after
every successful rename. The website writer may start only when the ledger is
`TARGET_ACTIVE` and the exact 14-name manifest is complete. Recovery resumes
an incomplete forward publication or reverses the recorded operations under
the same deployment lock; it never infers state from names alone.

## Failure and Rollback Rules

- Any failure before live publication removes only marker-owned temporary
  target collections and leaves the old application and collections active.
- Any failure after live rename but before source deletion restores the original
  names and old release under the same deployment lock.
- Any failure after source deletion restores the verified backup and old release
  while the website remains stopped and recovery remains suspended.
- Cleanup scripts must validate the exact database, collection allowlist,
  migration marker, source and target counts, release SHA, process identity, and
  deployment-lock ownership before a destructive command.
- A failed rollback never starts a writer against an unproven schema direction.
- Backup, migration, rename, deletion, and rollback evidence must not expose
  MongoDB URIs, service command lines, application secrets, or Cloudflare
  credentials.

## Application Architecture

Each of the 14 physical collections has one owning module and one explicit
persistence boundary. Existing Spring Data repositories that assume one Java
type per physical collection must be replaced or adapted behind domain stores.
Controllers and services continue to use domain types and must not depend on
the shared persisted envelope.

The canonical collection catalog and architecture tests must enforce:

- exactly 14 target collection names;
- exactly one owning module per target;
- every mapped kind appears once;
- no source collection name remains in runtime mappings or migration-exempt
  paths;
- no unscoped cross-kind Mongo query is present.

## Operational Interfaces

Production tooling must add:

- a read-only consolidation preview showing source/target mapping, counts,
  indexes, estimated bytes, and collision results;
- a guarded consolidation command requiring explicit confirmation;
- a restore command bound to the exact backup and migration marker;
- post-cutover inventory that reports target collection and per-kind counts.

Automatic deployment remains blocked while the consolidation marker is pending,
failed, or requires rollback.

## Validation Plan

### Automated

- Mapping, namespaced-ID, discriminator, and malformed-BSON unit tests.
- Repository contract tests proving every operation scopes by `_kind`.
- Index-definition tests for exact partial, unique, sparse, TTL, and collation
  behavior.
- Migration tests for all 48 source collections, empty optional sources,
  identifier collisions, stale targets, interrupted publication, and exact
  deletion allowlists.
- Real disposable MongoDB tests for full migration, rename, rollback, checksum,
  index, concurrency, and drop semantics.
- Architecture tests for the 14-name catalog and module ownership.
- Full Java, JavaScript, PowerShell 7, and required Windows PowerShell 5.1
  suites.
- Independent implementation and security reviews before publication.

### Production acceptance

- Verified backup path, size, SHA-256, and dry-run restore.
- Candidate database migration and alternate-port application smoke evidence.
- Before/after collection and index inventories.
- Per-kind document counts and canonical checksums before and after cutover.
- Exact active release and schema-direction marker.
- Local liveness/readiness and public root responses with status and body.
- MongoDB, website, and cloudflared service states.
- Proof that all superseded source and temporary collections are absent.
- Proof that automatic deployment uses the current protected tool bundle.

## Acceptance Criteria

The work is complete only when:

1. Production contains exactly the approved target collection set, subject only
   to MongoDB system collections.
2. Every pre-cutover document is represented exactly once under its target
   `_kind` and passes canonical checksum verification.
3. All required indexes and application behaviors pass automated and live
   verification.
4. Every superseded source collection has been dropped.
5. The verified backup and tested restore path are retained.
6. The new release and automatic-deploy tooling are active and healthy.

## Approved Decisions

- Physical target: approximately 15 collections; this design specifies 14.
- Cutover mode: one guarded maintenance window.
- Cleanup: immediate deletion of superseded collections after successful live
  verification.
- Design approach: domain-owned shared collections, not one universal
  collection and not a cosmetic inventory-only grouping.

## Open Questions

None. Implementation must stop and return to design review if a source
collection cannot preserve its ID, uniqueness, TTL, or query semantics within
the shared-collection contract.
