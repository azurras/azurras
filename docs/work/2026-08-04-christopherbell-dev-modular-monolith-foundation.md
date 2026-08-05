# christopherbell.dev Modular Monolith Foundation

- Status: `active`
- Owner/Agent: Codex primary agent with fresh task implementers and reviewers
- Started: 2026-08-04
- Related spec: [christopherbell.dev Modular Monolith](../specs/2026-08-04-christopherbell-dev-modular-monolith.md)
- Related implementation plan: [Modular Monolith Foundation](../implementation-plans/2026-08-04-christopherbell-dev-modular-monolith-foundation.md)

## Objective

Deliver the approved first foundation slice of the `christopherbell.dev` modular-monolith migration: a test-only Spring Modulith model, normalized ArchUnit dependency rules, a monotonic legacy-dependency baseline, and reviewable generated architecture documentation without changing the single-deployable runtime or any public application behavior.

## Background

The approved architecture prioritizes boundary safety and incremental capability slices. The heavily dirty authoritative spoke checkout must remain untouched, so implementation will use `codex/modular-monolith-foundation` in a clean sibling worktree created from refreshed `origin/main`. Each implementation task receives an independent specification-and-quality review before the next begins, followed by a broad branch review.

## Spoke Repositories

- `azurras/christopherbell.dev`
- Authoritative path: `A:\Projects\christopherbell.dev` (read-only for this initiative)
- Planned isolated worktree: `A:\Projects\christopherbell.dev-worktrees\modular-monolith-foundation`
- Branch: `codex/modular-monolith-foundation`

## Scope

- Execute all four tasks in the approved foundation implementation plan.
- Preserve one Spring Boot `website` JAR and the existing `website` plus `cbell-lib` Gradle topology.
- Keep Spring Modulith and ArchUnit enforcement on the test/build classpath only.
- Add deterministic, normalized, version-controlled architecture-debt enforcement.
- Prove full automated checks, packaged-JAR contents, and application behavior on an unused non-8080 port.
- Deliver through the repository's normal PR, CI, merge, protected Windows deployment, and production-verification workflow.

## Dispatched Tasks

- [Modular Monolith Foundation Implementation](../spoke-tasks/2026-08-04-christopherbell-dev-modular-monolith-foundation-implementation.md) - `active`
- Per-plan implementation briefs, implementer reports, and review packages will live in the isolated spoke's git-ignored `.superpowers/sdd/` workspace.

## Current State

The specification and executable plan are approved. Execution mode is subagent-driven development with a fresh implementer and task-scoped reviewer for each ordered task. Builder checkpoint creation and worktree preflight are in progress; no spoke source has been changed.

## Validation

- The implementation plan passed Builder mechanical validation and human execution review before approval.
- The plan records exact line-range edits, TDD evidence, full-check commands, boot JAR inspection, and alternate-port runtime verification.
- Current remote and worktree base will be revalidated before implementation.

## Blockers

None.

## Next Steps

1. Commit and push this Builder checkpoint.
2. Refresh the spoke remote, verify plan blocks against the current base, and create the isolated worktree.
3. Establish the per-plan SDD ledger and run the clean baseline.
4. Execute Tasks 1-4 serially with task review gates.
5. Complete branch review, runtime validation, PR/CI/integration, production verification, and Builder closeout.
