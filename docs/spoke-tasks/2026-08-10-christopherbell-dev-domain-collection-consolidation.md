# Dispatch: christopherbell.dev Domain Collection Consolidation

- Status: `closed`
- Work record: [Domain Collection Consolidation](../work/2026-08-10-christopherbell-dev-domain-collection-consolidation.md)
- Specification: [Domain Collection Consolidation](../specs/2026-08-10-christopherbell-dev-domain-collection-consolidation.md)
- Implementation plan: [Domain Collection Consolidation](../implementation-plans/2026-08-10-christopherbell-dev-domain-collection-consolidation.md)

## Target

- Repository: `azurras/christopherbell.dev`
- Registered authoritative path: `A:\Projects\christopherbell.dev`
- Isolated implementation path: `A:\Projects\christopherbell.dev-worktrees\domain-collection-consolidation`
- Branch: `codex/domain-collection-consolidation`
- Base: refreshed `origin/main` at `f4bc817d22abba70901fe4f17a93b4e52081085c`

## Objective

Execute the approved ten-task implementation plan through code, tests, review, PR/CI,
merge, guarded production cutover, immediate deletion of the exact superseded MongoDB
collection allowlist, runtime verification, and Builder closeout.

## Strict Scope

- Implement exactly the 14 target physical collections and 52-kind manifest in the plan.
- Preserve public behavior, authorization, BSON values, optimistic concurrency, TTL,
  unique/sparse/partial indexes, collation, and retention.
- Replace direct runtime collection ownership with the kind-scoped persistence boundary.
- Add manifest-driven preview/stage/verify/publish/drop/reverse/restore migration behavior.
- Generalize protected Windows schema-direction, deploy, writer-start, inventory, backup,
  candidate, rollback, and automatic-deploy gates for this cutover.
- Do not modify the authoritative checkout or unrelated dirty worktree state.
- Do not touch production until the merged release has passed every Task 8 gate.

## Required Engineering Practice

- Required skill: `write-jane-street-style-code` before any production source, test,
  reusable script, migration, executable configuration, or code-bearing template edit.
- Execute through `superpowers:subagent-driven-development`: one implementer, one
  task-scoped review, and reviewed fix loops before the next task.
- Use TDD: establish the intended RED, implement the minimum complete boundary, then run
  focused and proportionate regression evidence.

### Before-Edit Brief

Each task implementer must complete or refine these fields after read-only investigation
and before changing files:

- Behavior: the exact externally observable and persisted result for that plan task.
- Invariants: kind isolation, BSON/ID fidelity, index/concurrency semantics, and all
  unchanged domain/runtime properties the task must preserve.
- Boundary/API: service-facing ports, shared persistence interfaces, manifest entries,
  and production command contracts changed or consumed by the task.
- Effects and failures: reads/writes/process/service/database effects, ownership, exact
  fail-closed conditions, and redacted failure categories.
- Tests and evidence: first failing test, focused green command, real-boundary evidence
  when applicable, and regression command before commit.

## Validation

- Run the exact task-level commands in the plan and record discovered/passed/failed/skipped.
- Use private Gradle state and marker-owned disposable MongoDB roots/listeners.
- Run full `:website:test :website:bootJar`, full PowerShell 7 Pester, required Windows
  PowerShell 5.1 subsets, parser checks, disposable Mongo, restored production clone,
  alternate-port HTTP smoke, implementation review, and security diff review before PR.
- Require green CI/CodeQL and exact merge/deploy SHA before production cutover.
- For production, retain before/after inventories, per-kind counts/checksums/indexes,
  backup SHA/dry restore, URL/input/status/body, services, marker, and exact deletion proof.

## Expected Return Format

Each agent writes the full report to the SDD workspace file named in its dispatch and
returns only:

- `STATUS`: `DONE`, `DONE_WITH_CONCERNS`, `NEEDS_CONTEXT`, or `BLOCKED`
- `COMMITS`: exact commit SHA(s), or `none`
- `TESTS`: one-line commands and pass/fail/skip counts
- `CONCERNS`: one-line residual concerns or `none`

No agent may claim completion without a commit, current-head verification, self-review,
and the required report file. Production changes require the explicit Task 9 protected
command path and may not be approximated with ad hoc shell or Mongo commands.
