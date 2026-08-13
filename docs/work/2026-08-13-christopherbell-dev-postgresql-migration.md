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
- Implementation plan: not yet created; blocked on written-spec review
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

- The architecture, migration flow, recovery policy, pgAdmin access boundary,
  verification strategy, and completion criteria were approved interactively.
- No spoke implementation, database installation, schema creation, data copy, service
  mutation, or production cutover has started.
- The specification is ready for the user's written-artifact review.

## Blockers

No external blocker. Implementation planning waits for explicit approval of the saved
written specification.

## Validation State

- Builder and spoke context were inspected read-only.
- Current `origin/main` and the dirty authoritative spoke state were verified without
  modifying the spoke.
- PostgreSQL, Spring Boot SQL/jOOQ, pgAdmin, and supported-version choices were checked
  against current primary documentation during design.
- No implementation or runtime claims have been made.

## Next Steps

1. Review and approve the saved specification.
2. Create and validate a literal-file, line-range implementation plan from a clean
   isolated worktree refreshed from `origin/main`.
3. Execute the approved delivery loop through implementation, real-PostgreSQL testing,
   shadow rehearsals, PR/CI/merge, protected cutover, 14-day soak, MongoDB retirement,
   production verification, closure, and durable session memory.

