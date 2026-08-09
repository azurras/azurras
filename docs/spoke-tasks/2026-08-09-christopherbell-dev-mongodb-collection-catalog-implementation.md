# Implement christopherbell.dev MongoDB Collection Catalog

- Status: `closed`
- Work record: [christopherbell.dev MongoDB Collection Catalog](../work/2026-08-09-christopherbell-dev-mongodb-collection-catalog.md)
- Project spec: [MongoDB Collection Catalog](../specs/2026-08-09-christopherbell-dev-mongodb-collection-catalog.md)
- Implementation plan: [MongoDB Collection Catalog](../implementation-plans/2026-08-09-christopherbell-dev-mongodb-collection-catalog.md)
- Target repo: `azurras/christopherbell.dev`
- Authoritative local path: `A:\Projects\christopherbell.dev` (read-only)
- Isolated worktree: `A:\Projects\christopherbell.dev-worktrees\mongodb-collection-catalog`
- Branch: `codex/mongodb-collection-catalog`
- Base: `2f025762e248cab5befe0fb699e0560f57006572`

## Objective

Implement the approved catalog, drift enforcement, metadata-only inventory operation, and operator documentation without merging or mutating live MongoDB collections. Carry the result through automated and alternate-port local validation, task-scoped reviews, whole-branch review, PR and required CI, merge, protected deployment, metadata-only production verification, and Builder closeout.

## Required Skill and Before-Edit Brief

Required skill: `write-jane-street-style-code`.

Before changing production code, tests, scripts, code-bearing configuration, or copy-ready implementation examples, read and apply that skill. After read-only investigation and before the first edit, record a Before-Edit Brief covering:

- Behavior: the exact observable behavior being introduced or enforced.
- Invariants: especially one native MongoDB service, no application-document reads, no collection/data mutation, deterministic inventory ordering, and exact catalog-to-mapping agreement.
- Boundary/API: catalog rows, Java drift test, PowerShell functions, JSON payload, CLI command, Make target, and documentation boundaries.
- Effects and failures: Mongo shell invocation, protected configuration reads, stdout/stderr separation, validation failures, and non-zero exits.
- Tests and evidence: RED/GREEN focused tests, full automated suite, disposable Mongo validation, alternate-port packaged-app acceptance, and production metadata-only evidence.

Revise the brief if investigation changes any assumption, and include the final brief in the returned report.

## Strict Scope

- Follow the implementation plan exactly, including literal collection names, roles, statuses, PowerShell function signatures, JSON schema, CLI behavior, and validation sequence.
- Preserve the dirty authoritative checkout and make all edits only in the isolated worktree.
- Preserve unrelated checkout-time `gradlew.bat` line-ending changes; do not stage or commit them.
- Do not merge, rename, delete, compact, migrate, or write MongoDB collections or documents.
- Treat live-only collections as orphan candidates requiring separate backup-gated approval, never as automatically stale data.
- Do not weaken protected production ACLs or expose secrets.
- Use a private `GRADLE_USER_HOME`; do not impose short timeouts on Gradle or Pester.
- Validate a candidate on the plan's non-production port before any live listener change.

## Likely Files

- `docs/operations/mongodb-collection-catalog.md`
- `website/src/test/java/dev/christopherbell/configuration/MongoCollectionCatalogTest.java`
- `ops/production/windows/modules/Production.Operations.psm1`
- `ops/production/windows/prod.ps1`
- `prod.cmd`
- `Makefile`
- `ops/production/windows/tests/Production.Operations.Tests.ps1`
- `ops/production/windows/tests/Production.Command.Tests.ps1`
- `README.md`
- `docs/operations/windows-production.md`

## Validation Required

- Task-focused Java and Pester tests with recorded TDD RED/GREEN evidence.
- `:website:test`, `:website:jsTest`, `:website:build`, and applicable production Pester suites.
- Metadata inventory against a disposable MongoDB listener/database without application-document reads or writes.
- Packaged application acceptance on port `8097` against disposable data before production activity.
- Required GitHub PR checks and post-merge checks.
- Production command evidence that records database name, generation time, collection metadata, stats, and indexes while proving the site remains healthy and no mutation operation ran.

## Required Return

Return concise status plus durable report files containing commits and subjects, changed files, Before-Edit Brief, TDD RED/GREEN evidence, commands and exact results, review findings and fixes, PR/CI/merge state, deployment state, production verification, blockers or residual risks, and links needed for Builder ingestion and closure.

## Final Delivery

- Pull request: [#1352](https://github.com/azurras/christopherbell.dev/pull/1352)
- Merge commit: `0bcc8a9b83738df9c4adcf076e4be4443090448c`
- Test report: [MongoDB Collection Catalog Test Report](../test-reports/2026-08-09-christopherbell-dev-mongodb-collection-catalog-test-report.md)
- Spoke update: [Merged Delivery](../spoke-updates/2026-08-09-christopherbell-dev-mongodb-collection-catalog-merged-delivery.md)
- Review: [Branch Review](../spoke-reviews/2026-08-09-christopherbell-dev-mongodb-collection-catalog-branch-review.md)
- Outcome: reviewed, merged, deployed, production-verified, and ready for hub closure; no source GitHub issue existed.
