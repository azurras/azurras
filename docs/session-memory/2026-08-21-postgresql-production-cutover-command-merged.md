# 2026-08-21 - PostgreSQL Production Cutover Command Merged

## 12:50 - PostgreSQL Production Cutover Command Merged

### Request

Continue the approved christopherbell.dev PostgreSQL migration autonomously until the project is finished. Commit and push completed work, merge it after CI, preserve the dirty authoritative checkout, and pause only for genuine new authority.

### Project Context

Task 8 and the complete shadow rehearsal were already merged. Task 9 is the production persistence-authority transfer. Implementing and merging its guarded command is safe development work; executing it is materially different because it stops production, freezes the authoritative MongoDB source, finalizes PostgreSQL, publishes a one-way authority marker, stops MongoDB, and rotates the live website. That maintenance window requires separate explicit approval.

### Work Completed

- Created isolated worktree `A:\Projects\christopherbell.dev-worktrees\postgresql-cutover` from `origin/main` `bca4231b`; preserved the authoritative checkout's unrelated `M gradlew.bat` state.
- Added the public `prod.ps1 postgres-cutover -ConfirmPostgreSqlCutover` command and strict durable phase state machine.
- Added protected journal/evidence sidecars, release/database/catalog/target identity checks, explicit maintenance deadline checks, pre-authority Mongo recovery only after authenticated unlock proof, post-intent PostgreSQL-forward recovery, final Mongo and PostgreSQL archive/restore proof, candidate verification, environment/service authority publication, production verification, and 14-day/90-day soak evidence.
- Added the read-only `PostgresqlMigrationSourceSnapshotCli` and no-write live integration contract over all 52 kinds.
- Updated operations runbooks and exact architecture classification. V1-V27 migrations were unchanged.
- Committed the spoke as `88403a8d52dc455af442116dfc6502408976e16f`, pushed `codex/postgresql-cutover`, opened PR #1372, waited through all CI, and squash-merged to `main` as `ea6cead1a4fa14bd4ba3c5de65bb8dda91501d0c`.
- Refreshed `origin/main` explicitly and verified the merged cutover entry point.

### Decisions

- The persisted `AUTHORITY_PUBLICATION_STARTED` intent is the conservative one-way boundary. Any uncertainty after that record is resolved forward in PostgreSQL, never by restarting Mongo-backed writers.
- `-WhatIf` performs no preflight, lock, journal, process, database, or service effect; ordinary execution requires the exact confirmation switch.
- Production operators may not call Java `finalize` directly; the Windows wrapper is the sole supported production boundary.
- No new test report was created for Task 9 because the production app was intentionally not started or exercised as part of this implementation-only step. The prior Task 8 candidate report remains the latest runtime-app acceptance artifact; Task 9 runtime evidence will be recorded only during the approved maintenance window.

### Validation

- State machine 16/16; operations Pester 781 total with 753 passed and 28 expected skips.
- Read-only source snapshot 3/3 against MongoDB `/test` and PostgreSQL 18.4 `/test`, proving source and target shapes unchanged.
- Correct live migration package passed in 2m31s, including V1-V27, all-52 SHADOW/authenticated FINALIZE, failure injection, adapter query verification, and Mongo freeze/reader tests.
- Definitive local gate: BUILD SUCCESSFUL in 6m30s; website 2,268 tests, zero failures/errors, 75 expected skips; cbell-lib 123/123.
- PR CI: all nine checks green, including Windows/Ubuntu/macOS Java 25, jOOQ, Dependency Review, and all CodeQL languages.
- Cleanup: zero owned PostgreSQL schemas/history; Mongo `test` dropped; disposable ports 55434/57313 closed; exact worktree runtime and Gradle directories recycled. Production PIDs/listeners remained 8080/19812, 5432/7808, and 27017/5712.

### Current State

- Spoke `origin/main`: `ea6cead1a4fa14bdad963a4d33645b5bb61d88795c`.
- PR #1372: merged.
- Builder work remains `active` because production cutover and the post-cutover soak/retirement remain.
- Production is unchanged and available; no Task 9 production effect has run.

### Follow-ups

1. Obtain explicit approval for an up-to-30-minute production maintenance window.
2. Run the merged guarded cutover, capture exact backup/finalization/candidate/listener/runtime evidence, and save a local app test report.
3. Monitor 14 complete days of PostgreSQL production and restore evidence.
4. Only then execute Task 10 Mongo code/service/data retirement and close the project.
