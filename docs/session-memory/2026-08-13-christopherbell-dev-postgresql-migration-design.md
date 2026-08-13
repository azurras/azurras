# 2026-08-13 christopherbell.dev PostgreSQL Migration Design

## 09:52 - Approved PostgreSQL migration design checkpoint

### Request

Start the website's next database migration after completing the MongoDB collection
consolidation. Replace MongoDB with PostgreSQL, designate a PostgreSQL database named
`test` for local development and database-backed testing, and provide Compass-like visual
database viewer software for PostgreSQL.

### Project Context

- Builder is the workflow hub; the spoke is `azurras/christopherbell.dev`.
- The authoritative spoke checkout at `A:\Projects\christopherbell.dev` has extensive
  unrelated dirty state and was inspected read-only. Future implementation must use a
  clean isolated worktree from refreshed `origin/main`.
- Planning used `origin/main` at
  `e073823d14ffed0b4c113707d16c0ad0cfe1b7fa`.
- The completed Mongo consolidation provides 14 domain collections, 52 canonical kinds,
  126 manifest indexes, and explicit kind-scoped persistence boundaries.
- The Windows development host is production. No application, database, GUI, service,
  listener, or production-data mutation occurred during this design checkpoint.

### Work Completed

- Created the active Builder work ledger:
  `docs/work/2026-08-13-christopherbell-dev-postgresql-migration.md`.
- Created the ready-for-review project specification:
  `docs/specs/2026-08-13-christopherbell-dev-postgresql-migration.md`.
- Recorded the target platform, relational boundaries, migration catalog and engine,
  shadow rehearsals, cutover, failure handling, PostgreSQL recovery, monitoring, pgAdmin
  access, Mongo soak/retirement, verification, delivery phases, and acceptance criteria.
- Self-reviewed the specification for placeholders, contradictions, scope, and ambiguity;
  no unresolved design question remains.

### Decisions

- PostgreSQL becomes the sole durable database; no permanent hybrid remains.
- Use domain-relational schemas, Flyway versioned SQL, jOOQ-generated types, and explicit
  PostgreSQL adapters behind existing persistence ports.
- Run PostgreSQL 18 as a loopback-only native Windows service on the production host.
- Use repeated shadow imports followed by a stopped-writer final reconciliation; do not
  add production dual writes. Design against an assumed maximum 30-minute cutover.
- Use PostgreSQL database `test` for all local development and database-backed tests.
  Automated suites isolate through disposable uniquely named schemas inside `test`.
- Install pgAdmin 4 Desktop. Its `test` connection is read/write, while its production
  connection uses a database-enforced read-only viewer role. Privileged credentials are
  not registered in pgAdmin.
- Before the authority flip, fallback returns to untouched MongoDB. After PostgreSQL
  writes begin, recovery stays PostgreSQL-native and never switches silently to stale
  MongoDB.
- Stop and freeze MongoDB for a 14-day PostgreSQL soak, then remove Mongo runtime/service
  and exact live data after recovery evidence passes. Retain the final verified Mongo
  archive for 90 days with an exact evidence-gated deletion owner.

### Validation

- Verified Builder is clean on `main` with canonical `azurras/builder` origin before
  creating artifacts.
- Refreshed and inspected current spoke `origin/main` without touching the dirty checkout.
- Reviewed representative Mongo adapters, kind-scoped operations, Gradle dependencies,
  production configuration, and operational migration surfaces.
- Checked current primary documentation for PostgreSQL supported releases, Spring Boot
  SQL/jOOQ integration, jOOQ code generation, and pgAdmin Desktop capabilities.
- Placeholder scan found no unfinished markers, incomplete line ranges, or unresolved
  decisions in the work record or specification.
- No application tests or runtime checks were run because this checkpoint contains only
  Builder design artifacts and authorizes no implementation.

### Current State

- Architecture and all interactive design sections are approved.
- The saved written specification remains at the required user-review gate.
- No PostgreSQL installation, pgAdmin installation, schema migration, data copy, website
  edit, spoke branch, service change, or production cutover has begun.

### Follow-ups

1. Obtain explicit approval of the saved written specification.
2. Invoke the implementation-planning workflow and inspect a fresh spoke worktree for
   literal files and line ranges.
3. Review and validate the implementation plan before any production code or operational
   change.
4. After plan approval, execute the full Builder delivery loop through PR/CI/merge,
   protected cutover, 14-day soak, Mongo retirement, production verification, closure,
   and final archive-retention tracking.
