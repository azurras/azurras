# PostgreSQL cutover recovery review

## Status

complete

## Scope

- Work: [PostgreSQL migration](../work/2026-08-13-christopherbell-dev-postgresql-migration.md).
- Repository: `A:\Projects\christopherbell.dev-worktrees\postgresql-cutover`.
- Branch: `codex/fix-postgresql-cutover-evidence`.
- Reviewed commit: `07e2a48c`, including predecessor `36c2729d` and original `75cd4c5b`.
- Pull request: [1385](https://github.com/azurras/christopherbell.dev/pull/1385).

## Findings and disposition

No unresolved blocker in the scoped cutover correction. Independent reviewer
`review_migration_completion` approved final source inspection, subject to tests
and CI passing. This is not a claim of completed production migration.

Corrected blockers: mixed log/protocol stdout parsing; timestamp conversion
invalidating journal hashes; non-ISO writer lease; MongoDB service dependency;
stale persistent release authorization; and candidate startup recovery writes.
All five mutating startup handlers now skip the `deploy-smoke` profile. The VIN
cleanup moves from bean initialization to normal application readiness.

[Warning] Ordinary deployment still assumes MongoDB candidate backup/restore

Location: `ops/production/windows/modules/Production.Deploy.psm1:617`.
Contract: after PostgreSQL authority, normal deployment must validate the actual
backend without writing live application data or relying on stopped MongoDB.
Evidence: `Invoke-CandidateReleaseValidation` calls `New-ProductionBackup` and
`Restore-CandidateDatabaseFromBackup` regardless of backend.
Required change: deliver PostgreSQL-aware candidate and backup validation before
reenabling automatic deployment. Automatic deployment was disabled and read back
as disabled and not running on 2026-09-05; the website remained healthy.

## Validation

- Full Windows ops Pester suite: 795 passed, zero failed, 28 skipped.
- Focused Java suites: 79 passed, zero failures/errors, including ten real Spring
  profile dispatch checks and existing reconciliation/import handler tests.
- Witnessed failing regressions before correction, including four original
  startup handlers and separately the legacy VIN cleanup.
- `git diff --check`: passed. Private `.gradle-agent` cache excluded from commit.
- Production read-only probe: validated pre-authority ROLLED_BACK journal;
  default DateTime checksum mismatch versus preserved-string checksum match;
  five Mongo driver logs plus one valid snapshot record; catalog matched.
- Local readiness endpoint `http://127.0.0.1:8080/actuator/health/readiness`
  returned `{"status":"UP"}` after automatic-deployment pause.

## House-style and recovery review

Applied `write-jane-street-style-code`: exact protocol acceptance, preserved
digest/identity invariants, explicit side effects, service dependency readback,
database-enforced read-only candidate access, regression-first testing, and
forward-only authority recovery. Secrets are not embedded or printed.

## Remaining delivery gates

Final-commit CI, merge, protected release build, approved cutover, live endpoint
acceptance, PostgreSQL-aware ordinary deployment, and post-authority soak remain.
Preserve failed-attempt evidence; do not rewrite authority or bypass checksum
checks. Retire MongoDB only after the recorded fourteen-day soak and restore
proof; retain its final archive for ninety days.
