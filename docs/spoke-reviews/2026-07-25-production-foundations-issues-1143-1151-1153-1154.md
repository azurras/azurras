# Production Foundations Issues 1143, 1151, 1153, and 1154 Spoke Review

## Document Status

complete

## Reviewed Spoke

- Repository: `https://github.com/azurras/christopherbell.dev.git`
- Worktree:
  `A:\Projects\christopherbell.dev-worktrees\production-foundations-1143-1154`
- Branch: `codex/production-foundations-1143-1154`
- Base: `4b82116a0ed489c74eed144a478f1b3a3944ada2`
- Reviewed head: final staged pre-commit diff against the base; implementation
  commit and PR will be appended after publication.
- Issues: `#1143`, `#1151`, `#1153`, and `#1154`

## Scope Reviewed

- Production MongoDB URI and database profile configuration.
- Pre-refresh, redacted production setting validation and initializer
  registration in the packaged JAR.
- Typed mail configuration and password-reset enabled/disabled behavior.
- Protected Windows production environment parsing and service-launch allowlist.
- Root MongoDB Compose service, local defaults, and contributor commands.
- Atomic MongoDB lease document/service.
- Ordered migration interface, properties, record states, state transitions,
  runner, V001 indexes, checksum, and recovery documentation.
- Focused and full automated tests, disposable runtime evidence, cleanup, live
  production continuity, staged diff, and worktree hygiene.

## Findings

No remaining Blocker or Warning findings.

Review found and resolved two pre-publication consistency gaps:

1. An invalid `APP_MAIL_ENABLED` value originally returned from validation
   before the invalid sender joined the same report. The final initializer and
   protected environment parser aggregate the relevant setting names without
   echoing values; the focused regression proves the combined partition.
2. The V001 `migration_status_completed` index initially used ascending
   `completedAt`. It now matches the reviewed design with ascending `status` and
   descending `completedAt`.

The staged sensitive-value scan matched only the deliberately malformed
credential-bearing URI in the redaction regression. The value exists solely in
test input and the assertion proves it is absent from the thrown message. No
credential, token, private key, production secret, or disposable-runtime JWT is
staged.

## Validation Checked

- Focused RED compiled only after the new production settings, typed mail,
  lease, and migration types existed.
- Focused Pester RED failed 6 of 25 cases before `APP_MAIL_ENABLED` entered the
  allowlist and conditional validation.
- Final focused production-foundations Java suite: 32 passed.
- Packaged missing-settings start: exit `1`, no port `8090` listener, one
  `Invalid production configuration` report naming Mongo URI, JWT, sender, and
  provider key without values.
- Disposable first start and restart: `/` and readiness returned status code
  200; V001 remained exactly one `APPLIED` record with checksum
  `aec77e3e8cf68bf8d67f239ee0e842fbdad26ea9766ab04cbc3d74dd9ad93876`.
- Runtime Mongo metadata contained `migration_status_completed` and
  `lease_expiry`; the migration lease was unowned after both starts.
- Exact database `christopherbell_foundations_test_20260725230000` was dropped
  only after name and production-inequality checks; it no longer exists.
- Final `cleanTest + check`: 1,030 Java tests, 0 failures, 3 existing skips; 199
  JavaScript tests passed; `bootJar` and `verifySensorRuntime` passed.
- Full Windows production Pester: 247 total, 243 passed, 0 failed, 4 privileged
  or explicit-acceptance skips.
- `git diff --cached --check` passed.
- `application-prod.yml` contains no `mongodb://localhost:27017` fallback.
- Docker CLI is unavailable on this host; the structural Compose regression
  parsed the YAML and proved its image, port, volume, health check, and
  no-secret boundaries.
- Live production remained PID `29012`, port `8080`, status code 200 throughout
  the alternate-port run and final verification.

## House-Style Review

The implementation follows repository-native Java, Spring configuration,
PowerShell, YAML, and test conventions. Boundaries are explicit and narrow:
the initializer owns pre-refresh validation, `MailProperties` owns mail intent,
the lease service owns atomic query semantics, and the migration runner owns
ordering and durable state transitions. Tests partition absent, malformed,
disabled, enabled, contention, drift, incomplete, success, failure, restart,
and cleanup behavior. Error text carries setting names, migration IDs, and safe
categories only.

## Risks

- Migration lease duration is bounded at 2 minutes. V001 performs only
  idempotent index creation and completed within startup acceptance; future
  long-running migrations must be decomposed or introduce reviewed renewal
  orchestration before exceeding that lease.
- An interrupted migration deliberately blocks startup until an operator
  corrects the cause. The recovery runbook requires a backup, no active owner,
  exact-record inspection, and a bounded single-record action.
- Local Compose syntax could not be passed through the Docker CLI on this host.
  Cross-platform CI and the YAML structural regression remain required gates.
- Production migration execution has not occurred yet. Guarded deployment and
  production database inspection remain closure gates after merge.
- `gradlew.bat` has a checkout-only line-ending difference. It is unstaged and
  absent from the reviewed diff.

## Requested Changes

None remaining.

## Merge Readiness

Ready for an intentional spoke commit and PR after this Builder review
checkpoint is committed. Merge remains gated on Ubuntu, macOS, Windows,
Dependency Review, CodeQL, and guarded production acceptance.
