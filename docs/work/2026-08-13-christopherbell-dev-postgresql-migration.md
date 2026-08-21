# christopherbell.dev PostgreSQL Migration

- Status: `active`
- Owner context: Builder hub coordinating architecture, relational redesign,
  migration rehearsals, native-Windows production cutover, PostgreSQL acceptance,
  MongoDB retirement, and closeout
- Related spec: [PostgreSQL Migration](../specs/2026-08-13-christopherbell-dev-postgresql-migration.md)

## Objective

Replace MongoDB with PostgreSQL as the website's sole durable database, remodel the
current 52 persisted kinds into typed relational domain schemas, migrate every
production record through a rehearsed and reversible pre-cutover process, and retire
MongoDB only after PostgreSQL production and recovery evidence pass the approved soak.

## Owner and Scope

- Builder hub: `C:\Users\Christopher\Developer\builder`
- Spoke: `azurras/christopherbell.dev`
- Authoritative spoke path: `A:\Projects\christopherbell.dev` (preserve unrelated dirty state)
- Implementation surface: a clean isolated worktree refreshed from current `origin/main`
- Verified planning baseline: `origin/main` at `e073823d14ffed0b4c113707d16c0ad0cfe1b7fa`
- Current persistence baseline: 14 MongoDB domain collections, 52 canonical kinds,
  126 manifest-defined indexes, and explicit kind-scoped persistence boundaries
- Target: PostgreSQL 18 on native Windows, one production database named
  `christopherbell`, typed bounded-context schemas, Flyway migrations, jOOQ adapters,
  pgAdmin 4 Desktop, and no remaining MongoDB runtime dependency after retirement
- Development and database-test target: PostgreSQL database `test` only, using
  disposable schemas for automated-test isolation
- Cutover target: rehearsed shadow imports followed by a writer-stopped final
  reconciliation and an assumed maximum 30-minute maintenance window

## Related Artifacts

- Project specification: [PostgreSQL Migration](../specs/2026-08-13-christopherbell-dev-postgresql-migration.md)
- Prior MongoDB consolidation closure: [Domain Collection Consolidation](../work-closures/2026-08-12-christopherbell-dev-domain-collection-consolidation.md)
- Prior MongoDB consolidation work: [Domain Collection Consolidation](2026-08-10-christopherbell-dev-domain-collection-consolidation.md)
- Implementation plan: [PostgreSQL Migration Implementation Plan](../implementation-plans/2026-08-13-christopherbell-dev-postgresql-migration.md)
- Spoke task, reviews, test reports, production evidence, closure, and session memory:
  create during the approved delivery loop

## Approved Decisions

- PostgreSQL will be the sole durable production database; no permanent hybrid remains.
- The target is a domain-relational model, not a lift of Mongo envelopes into JSONB.
- Flyway owns versioned DDL and jOOQ provides explicit, generated, type-safe SQL access
  behind the existing domain persistence ports.
- PostgreSQL runs as a loopback-only native Windows service on the production host.
- Migration uses repeated shadow imports and reconciliation while MongoDB remains
  authoritative, followed by one stopped-writer authority transfer; there is no
  production dual-write period.
- Local development and every database-backed test use PostgreSQL database `test`.
- pgAdmin 4 Desktop is a required visual database viewer. Its production connection
  uses a database-enforced read-only role; privileged roles are never registered in it.
- Before authority transfer, rollback returns to untouched MongoDB. After PostgreSQL
  writes begin, recovery remains PostgreSQL-native and never discards new writes by
  silently switching back to stale MongoDB.
- MongoDB remains stopped and frozen for a 14-day soak. Its verified final archive is
  retained for 90 days before exact, evidence-gated deletion.

## Guardrails

- Do not modify the authoritative spoke checkout or its unrelated dirty state.
- Do not install PostgreSQL, pgAdmin, change a Windows service, touch production data,
  or edit website code until the written spec and implementation plan are approved.
- Invoke `write-jane-street-style-code` before production source, test, migration,
  reusable script, or executable configuration changes.
- Keep the website one Spring Boot deployable and preserve public HTTP, authorization,
  frontend, and externally visible domain behavior.
- Fail closed on unknown kinds, fields, identifiers, enum values, references, counts,
  hashes, constraints, indexes, roles, versions, databases, processes, paths, locks,
  services, or backup state.
- Never use H2 as a PostgreSQL substitute. Local and automated database work must
  verify the active database is exactly `test` before it starts.
- Do not register the owner, migrator, or website-runtime credentials in pgAdmin.
- Do not expose database credentials, connection strings containing secrets, service
  command lines, archive paths with secrets, or user data in logs or Builder evidence.
- Do not weaken protected production ACLs.
- Do not start a PostgreSQL writer until the frozen-source reconciliation, read-only
  candidate acceptance, and rollback-readiness gates all pass.
- After the PostgreSQL authority flip, do not automatically fall back to MongoDB.
- Do not remove MongoDB code, service state, or exact data until the 14-day retirement
  gate and PostgreSQL restore evidence pass.

## Current State

- Tasks 1 through 8 are implemented, reviewed, rehearsed, and merged through
  [PR #1370](https://github.com/azurras/christopherbell.dev/pull/1370) as
  `bca4231b4d36bdad963a4d33645b5bb61d88795c`.
- The Task 9 guarded production command is implemented, reviewed, and merged through
  [PR #1372](https://github.com/azurras/christopherbell.dev/pull/1372) as
  `ea6cead1a4fa14bd4ba3c5de65bb8dda91501d0c`; it has not been run in production.
- PostgreSQL 18/Flyway/jOOQ foundations, typed adapters for all 52 source kinds,
  the guarded migration engine, native Windows operations, and shadow/candidate
  verification are complete.
- Repeated live-source and 63,230-document restored-archive rehearsals reconciled all
  52 kinds. A PostgreSQL-only candidate passed the exact HTTP, role, scheduler,
  capacity, plan, latency, security, and cleanup matrix.
- Production Mongo, PostgreSQL 16, and the website listener remain unchanged. No
  FINALIZE, production authority marker, service dependency change, or cutover has run.

## Blockers

Task 9 requires an explicitly approved maintenance window before it can stop production
writers, take the final backup, transfer authority, or rotate the production listener.
This is an intentional authorization gate because the operation interrupts production
and the post-authority rollback direction is PostgreSQL-forward only.

## Validation State

- Definitive Task 8 verification passed 2,386 Java tests with zero failures/errors,
  plus JavaScript and PowerShell/Pester gates.
- The PostgreSQL candidate passed 39/39 exact HTTP statuses and latency budgets;
  role separation showed app sessions and zero bridge sessions.
- Final CI passed PostgreSQL 18 jOOQ generation, Java/JavaScript/Actions CodeQL,
  dependency review, and Java 25 builds on macOS, Ubuntu, and Windows.
- Task 9's definitive local gate passed 2,268 website tests and 123 cbell-lib tests
  with zero failures/errors. The guarded cutover state machine passed 16/16, operations
  Pester passed 753 with 28 expected skips, and the read-only source snapshot plus live
  migration package passed against disposable MongoDB and PostgreSQL 18.4 `test` databases.
- PR #1372 passed all nine GitHub checks and merged to refreshed `origin/main` as
  `ea6cead1a4fa14bdad963a4d33645b5bb61d88795c`.
- Independent review found no remaining Critical, Important, Blocker, or Warning
  finding in the Task 8 scope. The merged tree exactly matches the reviewed branch.
- The authoritative dirty spoke checkout was preserved throughout isolated worktree
  development, testing, PR integration, and merge.

## Next Steps

1. Obtain explicit approval for the up-to-30-minute Task 9 maintenance window.
2. Execute the merged `postgres-cutover -ConfirmPostgreSqlCutover` boundary, which owns
   the final backup, writer freeze, all-kind FINALIZE/reconciliation,
   alternate-port acceptance, one-way authority marker, and production listener rotation.
3. Monitor PostgreSQL for 14 full days and prove post-cutover restore before Task 10
   removes Mongo runtime code, service state, and exact live data.
