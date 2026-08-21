# christopherbell.dev PostgreSQL Shadow Rehearsal Merged

- Status: `active`
- Work record: [PostgreSQL Migration](../work/2026-08-13-christopherbell-dev-postgresql-migration.md)
- Test report: [PostgreSQL Shadow Rehearsal](../test-reports/2026-08-21-christopherbell-dev-postgresql-shadow-rehearsal-test-report.md)
- Spoke repository: `azurras/christopherbell.dev`
- Reporting task: Codex `/root`
- Isolated worktree: `A:\Projects\christopherbell.dev-worktrees\postgresql-migration`
- Authoritative checkout preserved: `A:\Projects\christopherbell.dev`

## Delivered

- Completed Tasks 1 through 8 of the approved PostgreSQL migration plan: PostgreSQL 18/Flyway/jOOQ foundation, typed relational schemas and adapters for all 52 source kinds, guarded migration and reconciliation, native Windows operations, and production-shadow rehearsal.
- Rehearsed repeatable SHADOW and RECONCILE operations against the read-only live Mongo source and a restored 63,230-document production archive.
- Verified a PostgreSQL-only application candidate with separated app and bridge roles, exact HTTP behavior, scheduler behavior, query plans, capacity, latency, security, and cleanup evidence.
- Added CI-owned PostgreSQL 18 jOOQ generation, exact-run generated-source artifacts, independent Java CodeQL code generation, and a 2 GiB website test worker so every supported runner executes the complete suite.
- Corrected the POSIX authority-test helper to retain owner directory traversal while preserving owner-only permissions.

## Commits and Pull Request

- Final branch head: `ced5b7cb1f1c9feb3c4e973a09fea7058ffbb497`.
- Pull request: [#1370 Complete PostgreSQL shadow rehearsal](https://github.com/azurras/christopherbell.dev/pull/1370).
- Squash merge on `main`: `bca4231b4d36bdad963a4d33645b5bb61d88795c`.
- The merged `main` tree is byte-equivalent to the final reviewed branch tree.

## Validation

- Definitive local gate: 2,386 Java tests, 2,311 passed, 75 expected skips, zero failures/errors.
- Candidate matrix: 39/39 exact statuses and 39/39 requests below 2,000 ms; maximum 75.298 ms.
- Restored archive: all 52 kinds and 63,230 documents reconciled repeatedly; live-source SHADOW replay and two RECONCILE digests were identical.
- Final PR checks passed: PostgreSQL 18 jOOQ generation; Java, JavaScript, and Actions CodeQL; dependency review; and Java 25 builds on macOS, Ubuntu, and Windows.
- Production remained unchanged; disposable candidates, databases, roles, processes, listeners, secrets, and temporary paths were removed.

## Residual and Next Action

Task 8 is complete. Task 9 is the guarded production authority transfer and remains gated by an explicitly approved maintenance window. Before the authority marker, rollback returns to untouched Mongo; after the marker and any PostgreSQL write, recovery must remain PostgreSQL-forward.
