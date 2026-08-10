# christopherbell.dev Domain Collection Consolidation

- Status: `active`
- Owner context: Builder hub coordinating design, implementation, production migration,
  immediate superseded-collection retirement, and closeout
- Related spec: [Domain Collection Consolidation](../specs/2026-08-10-christopherbell-dev-domain-collection-consolidation.md)

## Objective

Reduce the production `christopherbell` MongoDB database from 48 physical collections
to exactly 14 domain-owned collections while preserving every document and index
semantic, then delete all superseded collections during the same verified maintenance
cutover.

## Owner and Scope

- Builder hub: `C:\Users\Christopher\Developer\builder`
- Spoke: `azurras/christopherbell.dev`
- Authoritative spoke path: `A:\Projects\christopherbell.dev` (preserve unrelated dirty state)
- Implementation worktree: create from current `origin/main` before code edits
- Current deployed release: `f4bc817d22abba70901fe4f17a93b4e52081085c`
- Verified production baseline: 48 collections and 164 indexes on 2026-08-10
- Target: the 14 physical collections named in the approved specification
- Destructive boundary: immediate drop of the exact superseded allowlist is authorized
  only after candidate and live counts, checksums, indexes, catalog, runtime, and backup
  restore verification succeed under the protected deployment lock

## Related Artifacts

- Project specification: [Domain Collection Consolidation](../specs/2026-08-10-christopherbell-dev-domain-collection-consolidation.md)
- Prior narrow work: [Music Runtime State Consolidation](2026-08-09-christopherbell-dev-music-runtime-state-consolidation.md)
- Implementation plan: pending
- Spoke task, updates, review, test report, closure, and session memory: pending

## Current State

- The approved design maps all current domain data into exactly 14 physical collections.
- Shared documents use a kind discriminator, schema version, and lossless namespaced ID.
- The user approved one maintenance cutover and immediate deletion of superseded sources.
- No collection from this broader migration has been renamed or deleted yet.
- The website, MongoDB, cloudflared, automatic deployment, and protected writer-start
  boundary are healthy on the current release.

## Guardrails

- Use an isolated spoke worktree refreshed from `origin/main`; preserve the dirty
  authoritative checkout and unrelated worktree artifacts.
- Invoke `write-jane-street-style-code` before every production code, test, migration,
  script, or executable configuration edit.
- Prove the full migration against a disposable MongoDB and a restored production-data
  clone on an alternate application port before stopping the production writer.
- Fail closed on any unexpected database, collection, kind, BSON type, ID, count,
  checksum, index, schema marker, release, lock, process, path, or ACL state.
- Do not expose MongoDB URIs, service command lines, application secrets, or Cloudflare
  credentials in logs or evidence.
- Do not drop a source until the exact verified backup has passed a dry restore and the
  target release has passed live database and HTTP acceptance.

## Next Steps

1. Write, review, validate, commit, and push the literal implementation plan.
2. Create the isolated spoke worktree and execute the plan task by task with TDD.
3. Run independent implementation and security review, full automated verification,
   disposable MongoDB proof, and restored-production-clone alternate-port proof.
4. Merge through CI, perform the guarded maintenance cutover, immediately drop the exact
   superseded allowlist, and verify the 14-collection production catalog.
5. Save test, review, closure, and session-memory artifacts and close the hub work.
