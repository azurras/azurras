# Command-Center Configuration and Durable Power Actions

## Document Status

Ready for execution.

## Purpose

Resolve GitHub issues #1299, #1300, and #1301 by making command-center configuration fail fast, making fixed Windows command results trustworthy, and preserving pending machine power actions across application restarts.

## Background

The command center currently binds unvalidated durations, paths, counts, thresholds, byte limits, and ports. Its Windows executor hardcodes a 60-second shutdown delay and treats every non-cancellation process launch as successful immediately after `ProcessBuilder.start()`. Pending restart or shutdown state exists only in an `AtomicReference`, so restarting the website loses the UI state and permits a second power action while Windows still owns the first schedule.

Only comments authored by `azurras` may change scope or acceptance criteria. No issue comments alter these three issue bodies.

## Goals

- Reject invalid command-center configuration during Spring startup with property-specific constraint messages.
- Preserve safe relative defaults for simulated local development while requiring absolute fixed executable paths in Windows mode.
- Use the configured whole-second power delay in both API timestamps and the fixed `shutdown.exe /t` argument.
- Observe every fixed command for a bounded result interval, rejecting timeouts and nonzero exit codes.
- Persist exactly one pending machine power action atomically in MongoDB.
- Recover an unexpired pending action after application restart, remove elapsed state, and make cancellation idempotent.
- Keep executable selection and arguments closed over `CommandCenterActionType`; no request value may become a process argument.

## Non-Goals

- Changing authentication, challenge phrases, password confirmation, role checks, or power-action enablement.
- Persisting one-time challenges, failed-confirmation counters, or ordinary site-restart cooldowns.
- Adding arbitrary command execution, shell invocation, or user-provided command arguments.
- Querying undocumented Windows internals for shutdown state after the configured deadline.
- Supporting multiple simultaneous machine power actions.

## Requirements

### Configuration

- Add `@Validated` and field/nested validation to `CommandCenterProperties`.
- Validate non-null paths and modes, nonblank service/commit identifiers, port range 1-65535, positive bounded sampling/log/action values, and sensible percentage/temperature ranges.
- Add cross-field checks for history versus sample interval, provider timeout versus sample interval, CPU sensor process timeout versus refresh interval, per-actor challenge limit versus total challenge limit, whole-second power delay, and absolute executable paths when mode is `WINDOWS`.
- Add `command-result-timeout`, `max-challenges-per-actor`, and `max-challenges-total` action settings with safe shipped defaults.
- Keep the shipped local and production YAML valid; production remains Windows mode with fixed absolute executable paths and a 60-second delay.

### Windows execution

- Convert the configured power delay to an exact base-10 integer number of seconds and use it for `/t`.
- Run only direct `ProcessBuilder(List<String>)` commands built from the closed action enum and validated configuration.
- Wait at most `command-result-timeout` for each fixed command.
- Treat a completed zero exit as success; treat a nonzero exit or timeout as launch failure with an actionable, non-secret error.
- Destroy a still-running child after the timeout and preserve thread interruption as `IOException`.

### Durable pending state

- Store a singleton MongoDB document with a fixed identifier, action type, acceptance time, and execution deadline; do not persist actor credentials, challenge tokens, request content, or arbitrary arguments.
- Reserve the singleton atomically before launching a restart/shutdown. An unexpired existing record rejects a second power action.
- Roll back only the exact reservation when the fixed launch fails.
- Read the store for command-center snapshots so restarts before the deadline show the same pending action.
- Reconcile at application readiness and on reads by deleting state whose deadline has elapsed; at that point Windows owns or has completed the scheduled action.
- Cancellation with an active record runs fixed `shutdown.exe /a`, clears only that exact record after a successful zero exit, and audits the launch.
- Cancellation with no active record returns an accepted no-op result and audits an already-clear outcome, so retries are safe and do not launch `/a` again.

## Proposed Approach

Introduce a focused `CommandCenterPendingActionStore` backed by `MongoTemplate`. `reserve` uses a fixed `_id` and an atomic conditional upsert that can replace only absent/elapsed state; duplicate-key contention means another action is active. `clear` predicates on the action and execution deadline so a stale rollback or cancellation cannot erase a newer reservation. `active` removes elapsed state and returns only an immutable `PendingAction` projection.

Update `CommandCenterActionService` to remove its process-local pending reference. For power actions it derives the deadline from the validated property, reserves the durable state, records acceptance, invokes the fixed executor, and clears the exact reservation on failure. Snapshot reads and cancellation consult the durable store. An application-ready reconciler performs the same elapsed-state cleanup during startup.

Update `WindowsCommandExecutor` so every enum-selected direct command uses the configured result timeout. The command runner waits for completion, terminates a child that exceeds the bound, and returns the real exit code. The executor rejects all timeout/nonzero outcomes consistently. Its command builder formats only the validated integer delay; callers still supply only an enum.

## Files and Modules Involved

- `website/src/main/java/dev/christopherbell/admin/commandcenter/CommandCenterProperties.java`
- `website/src/main/java/dev/christopherbell/admin/commandcenter/action/WindowsCommandExecutor.java`
- `website/src/main/java/dev/christopherbell/admin/commandcenter/action/CommandCenterActionService.java`
- New focused pending-action document/store/reconciler classes under the command-center action package.
- `website/src/main/resources/application.yml` and `application-prod.yml`.
- Existing command-center property, executor, service, controller/snapshot, and application-context tests.
- New Mongo-backed store integration tests.

## Validation Plan

- Add configuration-context tests that prove representative lower/upper/cross-field failures and validate both shipped local and production profiles.
- Add fixed-command unit tests for a non-default power delay, bounded wait, zero exit, nonzero exit, timeout, interruption, and exact argument arrays.
- Add service tests for reservation-before-launch, duplicate rejection, exact rollback, restart recovery, expired reconciliation, successful cancellation, repeated cancellation, and cancellation failure retention.
- Add Mongo integration tests for atomic singleton reservation and stale-clear safety.
- Run targeted command-center tests first, then `:website:check` with an isolated Gradle user home.
- Start the packaged application with the production profile on a non-8080 port and isolated database; verify startup with shipped production configuration and health/root endpoints.
- After PR CI/CodeQL and merge, rotate production safely and verify root, public assets at the merge SHA, health probes, listener ownership, Mongo migration/index state, and Windows services.

## Rollback and Recovery

The application change can be reverted as one squash merge. The singleton collection is additive and contains no secret data; old application versions ignore it. A rollback may leave one elapsed or active document, but Windows remains the source of execution after launch and an operator can use the existing fixed `shutdown.exe /a` path if cancellation is required. No schema migration is needed because `_id` already enforces singleton uniqueness.

## Risks and Mitigations

- A service restart command may outlive the observation window: treat timeout as failure and audit it, while keeping the response-flush scheduler unchanged.
- A process could complete immediately after the timeout decision: terminate it before reporting failure and keep the window bounded.
- Mongo contention could produce duplicate-key exceptions: interpret only fixed-id duplicate contention as an active reservation, not a successful launch.
- A stale caller could clear newer state: predicate every clear on the exact action and deadline.
- Simulated local development uses relative executable defaults: absolute-path validation applies only to Windows mode.

## Open Questions

None. The user explicitly authorized continuation of the approved issue campaign without further routine approval.
