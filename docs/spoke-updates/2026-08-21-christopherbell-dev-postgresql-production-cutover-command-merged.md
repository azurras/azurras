# christopherbell.dev PostgreSQL Production Cutover Command Merged

- Status: `complete`
- Work record: [PostgreSQL Migration](../work/2026-08-13-christopherbell-dev-postgresql-migration.md)
- Spoke: `azurras/christopherbell.dev`
- Reporting context: Task 9 guarded cutover implementation and merge

## Changes

- Added the sole public `postgres-cutover -ConfirmPostgreSqlCutover` production authority-transfer command.
- Added a strict, tamper-evident phase journal and evidence sidecars spanning writer stop, final Mongo archive and dry restore, signed 52-kind finalization, reconciliation, PostgreSQL backup and dry restore, PostgreSQL-only candidate, one-way authority publication, production activation, verification, and soak entry.
- Added conservative recovery: Mongo is restored only before authority intent and only after authenticated `currentOp` proves it is unlocked; after intent, recovery is PostgreSQL-forward only.
- Added a read-only 52-kind source snapshot CLI that binds the frozen-source digest without database mutation or secret disclosure.
- Documented the maximum 30-minute maintenance window, exact phase chain, 14-day soak, and 90-day final Mongo archive retention.
- V1-V27 migration files remain unchanged.

## GitHub

- Branch commit: `88403a8d52dc455af442116dfc6502408976e16f`.
- Pull request: [#1372 Add guarded PostgreSQL production cutover](https://github.com/azurras/christopherbell.dev/pull/1372).
- Squash merge on `main`: `ea6cead1a4fa14bd4ba3c5de65bb8dda91501d0c`.

## Validation

- Task 9 state machine: 16/16 passed.
- Operations Pester: 781 total, 753 passed, 28 expected skips, zero failures.
- Live disposable source-snapshot contract: 3/3 passed against MongoDB `/test` and PostgreSQL 18.4 `/test`, with database-shape equality before and after.
- Correctly configured live migration package passed, including all-52 SHADOW/authenticated FINALIZE, V1-V27 schema/upgrade, query verification, failure injection, and Mongo freeze/reader tests.
- Definitive local gate: `BUILD SUCCESSFUL` in 6m30s; website 2,268 tests with zero failures/errors and 75 expected skips; cbell-lib 123/123.
- CI: jOOQ generation, Dependency Review, all CodeQL analyses, and Java 25 builds on Windows, Ubuntu, and macOS all green.
- Disposable PostgreSQL/MongoDB data, processes, ports, and private Gradle state were cleaned; production listeners remained unchanged.

## Authority Boundary and Next Action

The command is implemented and merged, but it has not been run against production. The up-to-30-minute maintenance window still requires explicit user approval because it stops the production writer, creates final archives, finalizes PostgreSQL, publishes a one-way authority marker, stops MongoDB, and rotates the live website. Task 10 remains gated on 14 complete days of successful PostgreSQL soak evidence.
