# ChristopherBell.dev ActivityPub Production Discovery Activation Review

## Document Status

complete

## Related Artifacts

- Work: [Production Discovery Activation](../work/2026-07-29-christopherbell-dev-activitypub-production-discovery-activation.md)
- Plan: [Production Discovery Activation Plan](../implementation-plans/2026-07-29-christopherbell-dev-activitypub-production-discovery-activation-plan.md)
- Test report: [Production Discovery Activation Test Report](../test-reports/2026-07-29-activitypub-production-discovery-activation.md)
- PR: [azurras/christopherbell.dev#1318](https://github.com/azurras/christopherbell.dev/pull/1318)

## Findings

No blockers or warnings.

## Reviewed Spoke

- Repository: `azurras/christopherbell.dev`
- Worktree: `A:\Projects\christopherbell.dev-worktrees\activitypub-production-activation`
- Branch: `codex/activitypub-production-activation`
- Branch commit: `1244dd31`
- Merged main SHA: `8405cd77d0f1743fe33d70cc80b47e37048090a0`

## Scope Reviewed

Reviewed the early Spring initializer, secret-path and file invariants, atomic creation and concurrent reuse, explicit-secret override, production/deploy-smoke profile boundary, production federation defaults, NodeInfo candidate/public deployment gates, operational documentation, and focused/full tests.

## Before-Edit Brief Conformance

- Behavior: production discovery starts with one stable secret; an explicit secret wins; disabled and non-production contexts perform no file I/O.
- Invariants: production accepts only the pinned ProgramData path, never replaces or truncates an existing key, requires a real 32-byte file under an existing real directory, and keeps inbound/outbound false with no peers.
- Boundary: the initializer supplies the existing configuration property before federation settings bind; only `deploy-smoke` may use an alternate absolute path.
- Effects/failures: randomness and atomic file creation are localized; unsafe, inaccessible, symbolic, malformed, or missing storage fails startup with redacted errors.
- Evidence: focused RED/GREEN tests, a full build, deployment tests, alternate-port restart, green multi-platform CI/CodeQL, automatic deployment, and production negative checks passed.

## House-Style Compliance

The change follows the Jane Street-style standard: explicit activation predicates, fail-closed path and file invariants, localized secret effects, deterministic error categories, no secret logging, concurrency tests, and configuration/runtime tests at the actual deployment boundary.

## Validation Checked

- Fresh local `:website:check`: 202 suites, 1,390 tests, zero failures/errors, three skipped.
- Fresh Pester deployment suite: 37 passed, zero failed/skipped.
- Alternate-port restart: root and NodeInfo 200; foreign WebFinger 404; inbox POST 403; 32-byte secret fingerprint unchanged.
- PR CI: Windows, Ubuntu, macOS, dependency review, and Actions/Java/JavaScript CodeQL passed.
- Production: listener rotated automatically; canonical/apex root and NodeInfo 200; foreign WebFinger 404; inbox POST 403; zero jobs, identities, eligible posts, and scan state; four native services Running/Automatic.

## Residual Scope

Outbound production delivery awaits a real operator-controlled peer. Inbound follows and interactions remain separate gates. NodeInfo software version currently reports `unknown`, a non-blocking metadata issue.

## Merge Readiness

Ready and verified after merge. PR 1318 rebased into main and the automatic production deployment passed with only read-only discovery enabled.
