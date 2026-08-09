# christopherbell.dev MongoDB Collection Catalog

- Status: `closed`
- Owner context: Builder hub coordinating subagent-driven implementation, review, publication, production rollout, and closure
- Related spec: [MongoDB Collection Catalog](../specs/2026-08-09-christopherbell-dev-mongodb-collection-catalog.md)
- Related implementation plan: [MongoDB Collection Catalog](../implementation-plans/2026-08-09-christopherbell-dev-mongodb-collection-catalog.md)

## Objective

Reduce MongoDB operational ambiguity without merging active data domains: establish a canonical catalog for the website's Spring Data MongoDB collections, enforce that catalog against mapped document types, and provide a metadata-only inventory command that can identify unmodeled live collections for later backup-gated review.

## Spoke Repository

- Repository: `azurras/christopherbell.dev`
- Authoritative path: `A:\Projects\christopherbell.dev` (read-only for this initiative because it contains unrelated user work)
- Isolated worktree: `A:\Projects\christopherbell.dev-worktrees\mongodb-collection-catalog`
- Branch: `codex/mongodb-collection-catalog`
- Starting revision: refreshed `origin/main` commit `2f025762e248cab5befe0fb699e0560f57006572`

## Scope

- Document every source-mapped physical MongoDB collection and its owning capability.
- Add an automated drift test that fails when mapped collection names and the catalog diverge.
- Add a read-only production inventory operation that returns collection metadata, statistics, and indexes without reading application documents.
- Document comparison and orphan-candidate handling.
- Do not merge, rename, migrate, delete, compact, or otherwise mutate MongoDB collections or data.
- Preserve the single native MongoDB service and the website's single-deployable runtime.

## Current State

Closed. The catalog and metadata-only inventory are implemented, independently reviewed, merged through [PR #1352](https://github.com/azurras/christopherbell.dev/pull/1352), deployed as `0bcc8a9b83738df9c4adcf076e4be4443090448c`, and production-verified. The live inventory contained 47 collections and 163 indexes, with no live-only name and four cataloged-but-uncreated names. No collection or document was mutated.

## Blockers

None.

## Validation

- Specification checkpoint: Builder commit `95eb2da`.
- Implementation-plan checkpoint: Builder commit `80ac3441ce981f9c87306ced24b5a1105d6f8d49`.
- Test report checkpoint: Builder commit `723e403`.
- Spoke update checkpoint: Builder commit `77ccab8`.
- Spoke review checkpoint: Builder commit `b89090e`.
- Final local `:website:check`: 1,679 Java tests and both 83-test production suites passed; `BUILD SUCCESSFUL in 5m 7s`.
- PR and main Ubuntu/macOS/Windows, Dependency Review, and CodeQL gates passed.
- Production PID rotated from `13484` to `62412`; exact merge SHA, local/public HTTP 200 responses, Running/Automatic services, and metadata-only MongoDB inventory were verified.

## Follow-Ups

- Use the catalog and `prod.cmd mongo-inventory` for future ownership/drift review.
- Treat the four cataloged-but-uncreated names as unexercised flows, not cleanup candidates.
- Require separate explicit approval, compressed backup, restore validation, impact reporting, rollback retention, and one-at-a-time verification before any future collection cleanup or consolidation.
