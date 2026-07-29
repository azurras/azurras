# christopherbell.dev Performance, Scalability, and Library Optimization

- Status: active
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

- Project spec: [christopherbell.dev Performance, Scalability, and Library Optimization](../specs/2026-07-29-christopherbell-dev-performance-scalability-library-optimization.md) (`ready-for-review`).
- Related active campaign: [Complete christopherbell.dev Issues 1258-1307](2026-07-29-complete-christopherbell-dev-issues-1258-1307.md).
- Implementation plan: pending written-spec review and approval.

## Spoke Repositories

- `christopherbell-dev`: authoritative checkout `A:\Projects\christopherbell.dev`;
  it was dirty, ahead 3, and behind 93 during the design review and remains
  untouched.
- Evidence checkout:
  `A:\Projects\christopherbell.dev-worktrees\repository-documentation-coverage-20260729`
  at refreshed `origin/main` commit
  `5de2a8b02941ff7e95b6f2648b7bada9397f68b9`.

## Dispatched Tasks

No external agent tasks were dispatched. The primary agent performed the
read-only investigation.

## Current State

- The user approved a hot-path-first approach with extraction only after reuse
  is proven.
- The user approved the backend, browser, Java-library, and verification design
  sections.
- Newly validated hot paths include static-asset authentication database work,
  browser-session read/write amplification, conversation unread-count N+1
  queries, and an unbounded VIN rate-limit bucket map.
- The global browser bootstrap reaches 14 modules totaling 172,868 raw bytes and
  3,956 source lines; `main.css` is 154,946 raw bytes and contains multiple
  feature-exclusive sections.
- Stable cursor and Mongo lease primitives have multiple website consumers and
  qualify for `cbell-lib`.
- Repeated WFL UI, alert, URL-trimming, and sanitization behavior qualifies for
  focused `static/js/lib` consolidation where semantics match.
- Forty-three issues from #1258-#1307 remain open; many already own feed, WFL,
  Shared Folder, and build scaling work and will not be duplicated.

## Validation

- Refreshed `origin/main` and confirmed the reviewed clean commit.
- Inspected repository status, architecture instructions, recent history,
  module dependencies, largest source files, Mongo access patterns, scheduled
  jobs, local stores, HTTP clients, browser imports, CSS size, live static cache
  headers, and all current GitHub issue titles and relevant historical issues.
- No spoke source file, database, process, service, listener, issue, or pull
  request was changed during the investigation.
- Builder artifact indexes and hub validation will be refreshed before this
  planning checkpoint is committed.

## Blockers

- The written specification requires user review before implementation planning.
- Spoke implementation is not authorized by the design approvals recorded so
  far and requires a separate explicit user instruction.

## Next Steps

1. Ask the user to review the written specification.
2. After approval, create and validate a literal line-range implementation plan
   against refreshed current mainline.
3. Ask for explicit execution authorization after the implementation plan is
   reviewed.
4. If authorized, execute independent optimization batches with focused tests, full
   `:website:check`, alternate-port runtime acceptance, pull-request CI, merge,
   and production-safe verification.
5. Record Builder test, update, review, session-memory, and closure artifacts as
   implementation proceeds.
