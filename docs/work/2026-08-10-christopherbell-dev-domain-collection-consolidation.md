# christopherbell.dev Domain Collection Consolidation

- Status: `closed`
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
- Implementation plan: [Domain Collection Consolidation](../implementation-plans/2026-08-10-christopherbell-dev-domain-collection-consolidation.md)
- Spoke task: [Domain Collection Consolidation](../spoke-tasks/2026-08-10-christopherbell-dev-domain-collection-consolidation.md)
- Spoke update: [Merged and Production-Verified Delivery](../spoke-updates/2026-08-12-christopherbell-dev-domain-collection-consolidation-completion.md)
- Spoke review: [Final Delivery Review](../spoke-reviews/2026-08-12-christopherbell-dev-domain-collection-consolidation-review.md)
- Test report: [Production Test Report](../test-reports/2026-08-12-christopherbell-dev-domain-collection-consolidation-test-report.md)
- Closure: [Domain Collection Consolidation Closure](../work-closures/2026-08-12-christopherbell-dev-domain-collection-consolidation.md)
- Session memory: [Domain Collection Consolidation](../session-memory/2026-08-12-christopherbell-dev-domain-collection-consolidation.md)

## Final State

- Production contains exactly the approved 14 physical collections, 52 canonical kinds,
  and 126 manifest-defined indexes; every inventory compliance check is true.
- All superseded sources were removed by the guarded cutover after backup, candidate,
  checksum, index, and stopped-writer verification.
- Release `62e1c7193414ecab266a217d221141120c8ecaef` is live; the website, MongoDB,
  and cloudflared services are Running and local/public health checks return HTTP 200.
- The final read-only inventory load-order repair merged in PR #1369 and may ride the
  next ordinary deployment; it is not required for current application or schema health.

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

None required. Retain the verified backup and normal monitoring; deploy PR #1369 with a
future ordinary release rather than forcing an otherwise unnecessary production restart.
