# christopherbell.dev Music Runtime State Consolidation

- Status: `active`
- Owner context: Builder hub coordinating design, implementation, production-safe migration,
  observation, and separately approved retirement
- Related spec: [Music Runtime State Consolidation](../specs/2026-08-09-christopherbell-dev-music-runtime-state-consolidation.md)

## Objective

Safely reduce the website's MongoDB collection count by consolidating the compatible
`music_queue_state` and `music_radio_state` singleton collections into one
`music_runtime_state` collection without coupling their concurrency, changing public
music behavior, or deleting rollback data during the cutover.

## Owner and Scope

- Builder hub: `C:\Users\Christopher\Developer\builder`
- Spoke: `azurras/christopherbell.dev`
- Authoritative spoke path: `A:\Projects\christopherbell.dev` (preserve unrelated dirty state)
- Current deployed baseline: `0bcc8a9b83738df9c4adcf076e4be4443090448c`
- Scope: music queue and radio runtime state only
- Destructive boundary: no collection drop is authorized by design approval or spec approval

## Related Artifacts

- Project specification: [Music Runtime State Consolidation](../specs/2026-08-09-christopherbell-dev-music-runtime-state-consolidation.md)
- Prior collection catalog work: [MongoDB Collection Catalog](2026-08-09-christopherbell-dev-mongodb-collection-catalog.md)
- Implementation plan: [Music Runtime State Consolidation](../implementation-plans/2026-08-09-christopherbell-dev-music-runtime-state-consolidation.md)
- Test report, spoke update, review, and closure: pending execution

## Current State

- A fresh metadata-only production inventory reported 47 live collections and 163 indexes.
- No live collection is unowned or absent from the collection catalog.
- The user selected active-collection consolidation for a simpler domain model, beginning
  with music runtime state.
- `music_queue_state` and `music_radio_state` are compatible singleton-state collections:
  each currently contains one `_id: "global"` document and only the `_id` index.
- The approved architecture uses one collection with separate queue and radio documents,
  preserving independent optimistic-lock versions.
- The approved rollback-retention window is seven days.
- Verification contract: full Pester on PowerShell 7 plus focused Music runtime, command, and
  operations tests on Windows PowerShell 5.1. The untouched base has 85 unrelated PS5-only
  incompatibility failures and they are recorded rather than added to this migration scope.

## Guardrails

- Use an isolated spoke worktree refreshed from `origin/main`; do not modify the
  authoritative dirty checkout.
- Prove the migration and website behavior against a production-data clone on an
  alternate port before production cutover.
- Fail closed on unexpected counts, IDs, shapes, versions, or partial destination state.
- Leave both source collections intact during the seven-day observation window.
- Treat retirement as a separate destructive phase requiring a fresh backup, checksum,
  disposable restore proof, literal-name preview, and explicit user approval.
- Never weaken production ACLs or expose document values in operational evidence.

## Next Steps

1. Obtain user review and approval of the written project specification.
2. Write, review, validate, commit, and push a literal implementation and rollback plan.
3. Execute the non-destructive consolidation cutover through normal spoke delivery.
4. Observe production for seven days.
5. Return with exact retirement evidence and request approval before dropping the two
   legacy source collections.
