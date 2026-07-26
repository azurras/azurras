# Production Foundations Issues 1143, 1151, 1153, and 1154 Test Report

## Document Status

complete

## Story/Issue

GitHub issues `azurras/christopherbell.dev#1143`, `#1151`, `#1153`, and
`#1154`.

## Branch

Spoke branch `codex/production-foundations-1143-1154` from `origin/main`
commit `4b82116a0ed489c74eed144a478f1b3a3944ada2`. The branch has not yet been
published or merged; PR, CI, merge, issue closure, and production evidence will
be appended after the local checkpoint.

## App / Environment

- App: `christopherbell.dev` Spring Boot application.
- Worktree:
  `A:\Projects\christopherbell.dev-worktrees\production-foundations-1143-1154`.
- JDK: Eclipse Temurin `25.0.3`.
- MongoDB: native server `8.3.2` at `mongodb://127.0.0.1:27017`.
- Packaged acceptance profiles: `prod,deploy-smoke` on alternate port `8090`.
- Disposable database:
  `christopherbell_foundations_test_20260725230000`.
- Mail acceptance setting: `APP_MAIL_ENABLED=false`; no sender or provider key
  was supplied.
- JWT acceptance setting: a synthetic 32-plus-character value used only in the
  disposable process environment.
- Production safety: live listener PID `29012` remained on port `8080` and `/`
  returned `200` before, during, after, and at final verification. Only verified
  candidate PIDs `38736` and `48956` were stopped on port `8090`.

## Local Run Details

The packaged JAR was built with:

```powershell
.\gradlew.bat :website:bootJar --no-daemon --no-watch-fs --max-workers=1 --console=plain
```

The missing-settings acceptance started `website.jar` with profiles
`prod,deploy-smoke` and port `8090` after removing the production setting
variables. The valid runtime acceptance supplied:

```text
APP_MAIL_ENABLED=false
SPRING_MONGODB_URI=mongodb://127.0.0.1:27017
SPRING_MONGODB_DATABASE=christopherbell_foundations_test_20260725230000
```

The synthetic JWT value is intentionally omitted from this report. Runtime
acceptance occurred on `2026-07-25` between approximately `22:55` and
`23:00 -05:00`. Final full verification completed shortly afterward.

## Test Cases

1. Prove focused RED failures for missing production initializer, typed mail,
   Compose, lease, and migration contracts.
2. Validate that production MongoDB configuration has no localhost URI fallback
   and reads `SPRING_MONGODB_URI` explicitly.
3. Validate all missing production settings in one pre-refresh redacted report.
4. Validate malformed Mongo URI, weak JWT, malformed mail switch, invalid sender,
   and placeholder provider-key partitions without echoing supplied values.
5. Validate explicit mail disablement without sender/provider credentials and
   prove password-reset delivery does not resolve the sender.
6. Validate the production PowerShell environment allowlist, Boolean switches,
   conditional mail requirements, placeholder rejection, and aggregate errors.
7. Parse the root Compose YAML and prove MongoDB `8.3.2`, loopback-only port,
   persistent named volume, health check, and absence of application secrets.
8. Validate atomic lease acquire/contention/renew/release queries.
9. Validate stable migration ordering, duplicate-ID rejection, lease
   contention, applied skip, checksum drift, incomplete-state failure, safe
   failure recording, owner-scoped state transitions, and bounded properties.
10. Recompute and assert V001 checksum
    `aec77e3e8cf68bf8d67f239ee0e842fbdad26ea9766ab04cbc3d74dd9ad93876`.
11. Start the packaged production-profile JAR without required settings and
    prove it exits before binding port `8090`.
12. Start and restart the packaged JAR against the exact disposable database;
    inspect migration records, indexes, and lease state after both starts.
13. Stop only the verified candidate, drop only the validated disposable
    database, prove it is absent, and recheck live port `8080`.
14. Run the full Gradle check lifecycle, JavaScript suite, Windows production
    Pester suite, and `git diff --check`.

## Data Sent

- Local HTTP `GET` requests to `http://127.0.0.1:8090/` and
  `/actuator/health/readiness` during both disposable starts.
- Local HTTP `GET` requests to `http://127.0.0.1:8080/` for production
  continuity only.
- MongoDB reads of the exact disposable database's
  `application_migrations` and `application_leases` collections and index
  metadata.
- One exact `dropDatabase()` operation after validating the disposable database
  name against `^christopherbell_foundations_test_[0-9]{14}$` and proving it was
  not `christopherbell`.
- No production database mutations, account credentials, mail credentials,
  GitHub attachments, or untrusted issue instructions were used.

## Response Received

- Missing-settings packaged start: process exit `1`; port `8090` remained
  unbound.
- The report header was `Invalid production configuration` and named exactly the
  required missing boundaries: `SPRING_MONGODB_URI`, `APP_JWT_SECRET`,
  `APP_MAIL_FROM`, and `RESEND_API_KEY`. Configured values were not present.
- First disposable start: PID `38736`; the running app response status code:
  200 OK for `/` and 200 OK for readiness.
- First Mongo inspection: exactly one migration record with ID
  `001-ensure-migration-infrastructure`, status `APPLIED`, and the reviewed
  checksum.
- Index inspection: `application_migrations` contained `_id_` and
  `migration_status_completed`; `application_leases` contained `_id_` and
  `lease_expiry`.
- Lease inspection: `application-migrations` had no owner after startup.
- Restart against the same database: PID `48956`; the running app response
  status code: 200 OK for `/` and readiness; V001 remained exactly one
  `APPLIED` record; the lease remained unowned.
- Cleanup: PID `48956` stopped, port `8090` was free, `dropDatabase()` returned
  `ok: 1`, and the exact database no longer appeared in `listDatabases`.
- Live production continuity: PID `29012` and `/` status `200` throughout.

## Pass / Fail

- PASS: production Mongo URI is environment-driven and missing configuration
  fails before context refresh or port binding.
- PASS: production configuration errors aggregate setting names and redact
  values.
- PASS: mail is explicit; production defaults to enabled for compatibility and
  can be intentionally disabled without resolving a sender.
- PASS: local MongoDB has a pinned, persistent, loopback-only Compose contract.
- PASS: migration IDs/checksums are immutable, ordered, leased, durably
  recorded, idempotently restartable, and fail closed on drift or incomplete
  state.
- PASS: recovery documentation is backup-first and limits destructive action to
  one exact incomplete record.
- PASS: temporary runtime data and port `8090` were cleaned up without affecting
  production.

## Evidence

- Focused Java RED: compile failed on the intentionally missing
  `ProductionSettingsApplicationContextInitializer`, `MailProperties`, lease,
  and migration types.
- Focused Pester RED: 6 of 25 failed because `APP_MAIL_ENABLED` was not yet
  allowlisted or validated.
- Final focused Java suite: 32 tests passed across production settings, mail,
  Compose, leases, and migrations.
- Final full Gradle command:
  `.\gradlew.bat :website:cleanTest :website:check --no-daemon --no-watch-fs --max-workers=1 --console=plain`.
- Final Java result: 1,030 tests, 0 failures, 3 existing skips.
- Final JavaScript result: 199 passed, 0 failed.
- Final Windows Pester result: 247 total, 243 passed, 0 failed, 4 skipped.
- Pester skips: one installed-worker probe requires the exact
  `CBDEV_INSTALLED_WORKER_ACCEPTANCE` phrase; three protected NTFS integration
  tests require an elevated PowerShell session. These are environment/authority
  gates, not product failures.
- `git diff --check`: passed.
- Docker CLI: not installed on this host, so `docker compose config` was not
  executable; the checked YAML structural test passed in the Java suite.

## Bugs / Follow-ups

- The baseline command-center timing race recurred once on the first macOS PR
  run. Commit `4e767dfd87a03f873114d496600f1a68d8f560c6` replaced the provider-side
  pre-return latch with a single-worker executor barrier, which proves the
  timed-out service wrapper has actually completed before resubmission. The
  exact method, all 12 owning tests, and the full 1,030-test local suite passed;
  the fresh macOS, Ubuntu, and Windows CI matrix passed on that commit.
- The worktree retains the checkout-only `gradlew.bat` line-ending difference.
  It must remain excluded from the spoke commit.
- Docker remains unavailable on this host; the checked structural Compose test
  is the acceptance evidence for that contract.

## Publication and Closure

- Implementation commit: `257e2f656c030aa585b99cb07d58d96489a980b4`.
- CI race repair: `4e767dfd87a03f873114d496600f1a68d8f560c6`.
- [PR #1252](https://github.com/azurras/christopherbell.dev/pull/1252)
  passed Dependency Review, CodeQL for Actions, Java/Kotlin, and
  JavaScript/TypeScript, plus Java 25 builds on Ubuntu, macOS, and Windows.
- PR #1252 squash-merged to `main` as
  `965b25bb3e703a2e67a5064d777a9ab1998f26a1` on 2026-07-25 local time.
- Issues #1143, #1151, #1153, and #1154 closed automatically from the merged
  PR.
- Guarded production deployment replaced Java listener PID `29012` with
  `30976`. During startup readiness briefly returned 503 as expected, then
  reached 200; `/` remained 200 and the Windows service remained Running.
- Read-only production Mongo inspection found exactly one APPLIED
  `001-ensure-migration-infrastructure` record with checksum
  `aec77e3e8cf68bf8d67f239ee0e842fbdad26ea9766ab04cbc3d74dd9ad93876`,
  the `migration_status_completed` and `lease_expiry` indexes, and a released
  `application-migrations` lease with no owner token and epoch expiry.
- The deployed HTML references assets under the exact merge SHA namespace
  `965b25bb3e703a2e67a5064d777a9ab1998f26a1`.

Overall result: PASS. No remaining batch blocker or acceptance gap.
