# christopherbell.dev Performance, Scalability, and Library Optimization

- Status: closed
- Owner/Agent: Codex primary agent
- Started: 2026-07-29

## Objective

Design an evidence-backed roadmap for website speed, resource-bounding, and
horizontal-scaling improvements while identifying stable shared Java behavior
for `cbell-lib` and stable shared browser behavior for `static/js/lib`. Deliver
spoke changes only after separate explicit user authorization.

## Scope

- Measure and optimize common backend and browser hot paths.
- Remove authentication work from static assets.
- Reduce browser-session database operations without weakening invalidation.
- Batch newly identified N+1 query paths and bound local/upstream resource use.
- Split global browser dependencies and CSS by actual route ownership.
- Correct the Java and browser library boundaries.
- Reuse the active #1258-#1307 campaign for already-tracked scaling work.

## Related Specs and Plans

- Project spec: [christopherbell.dev Performance, Scalability, and Library Optimization](../specs/2026-07-29-christopherbell-dev-performance-scalability-library-optimization.md) (approved).
- Related active campaign: [Complete christopherbell.dev Issues 1258-1307](2026-07-29-complete-christopherbell-dev-issues-1258-1307.md).
- Implementation plans:
  - [Authentication Request Efficiency](../implementation-plans/2026-07-29-christopherbell-dev-authentication-request-efficiency.md)
  - [Backend Query and Resource Bounds](../implementation-plans/2026-07-29-christopherbell-dev-backend-query-resource-bounds.md)
  - [Browser Delivery Optimization](../implementation-plans/2026-07-29-christopherbell-dev-browser-delivery-optimization.md)
  - [Shared Library Boundaries](../implementation-plans/2026-07-29-christopherbell-dev-shared-library-boundaries.md)

## Spoke Repositories

- `christopherbell-dev`: authoritative checkout `A:\Projects\christopherbell.dev`;
  it was dirty, ahead 3, and behind 93 during the design review and remains
  untouched.
- Final planning checkout:
  `A:\Projects\christopherbell.dev-worktrees\performance-planning-20260729`
  at refreshed `origin/main` commit
  `f31535f29312d24573a6031b0162aa8ebc4b5318`.
- The earlier evidence checkout acquired unrelated documentation changes during
  planning and was preserved without cleanup or reuse.

## Dispatched Tasks

No external agent tasks were dispatched. The primary agent performed the
read-only investigation.

## Current State

- Authentication request-efficiency work is merged and production-verified through
  PR #1329 at commit `3a8e249a45e50e53f1ddc6fa1c520dcc82adee03`.
- Backend query/resource-bounds work is merged and production-verified through
  PR #1336 at commit `c4d60ce0c92281c201d063cfd6a07563f4a7b230`. See
  [Backend Query and Resource Bounds Test Report](../test-reports/2026-08-01-christopherbell-dev-backend-query-resource-bounds.md).
- Browser delivery work is merged and production-verified through PR #1337 at
  commit `95d805658beaa4c62a8b5e56af9bbf1c0aca66a6`. The global JavaScript graph
  fell from 174,433 bytes across 14 modules to 64,123 bytes across 10 modules;
  feature JavaScript and CSS are demand-loaded and static assets use content
  fingerprints. See [Browser Delivery Optimization Test Report](../test-reports/2026-08-01-christopherbell-dev-browser-delivery-optimization.md).
- Shared-library boundary work is merged and production-verified through PR
  #1338 at commit `2b40bd860d9e4e05aa18b4dd63e13a390d41208e`.
  Stable cursors and generic Mongo leases now live in `cbell-lib`, `TestUtil` is
  test-fixture-only, JJWT is website-owned, and the workflow engine is WFL-owned.
  See [Shared Library Boundaries Test Report](../test-reports/2026-08-01-christopherbell-dev-shared-library-boundaries-test-report.md).
- All four approved implementation plans are complete. Their local gates,
  independent reviews, required PR checks, post-merge CI/CodeQL, automatic
  deployments, public smoke tests, and cleanup checks passed.

## Validation

- Refreshed `origin/main` and confirmed the reviewed clean commit.
- Inspected repository status, architecture instructions, recent history,
  module dependencies, largest source files, Mongo access patterns, scheduled
  jobs, local stores, HTTP clients, browser imports, CSS size, live static cache
  headers, and all current GitHub issue titles and relevant historical issues.
- No spoke source file, database, process, service, listener, issue, or pull
  request was changed during the investigation.
- All four implementation plans pass the checked-in implementation-plan
  validator; Builder artifact indexes and hub validation are refreshed before
  this planning checkpoint is committed.

## Blockers

- None.

## Next Steps

No campaign work remains. Future optimization should begin with fresh production
measurements and reuse the existing issue backlog rather than reopening these
completed implementation plans.
