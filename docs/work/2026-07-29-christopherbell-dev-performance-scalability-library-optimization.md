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
- The user approved the written specification. Four ordered implementation
  plans now pass Builder's literal Code Edit validation and human
  execution-readiness review.

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

- Spoke implementation is not authorized by the design approvals recorded so
  far and requires a separate explicit user instruction.

## Next Steps

1. Ask for explicit execution authorization and the preferred execution mode.
2. If authorized, execute the authentication and backend plans first, the
   browser plan independently, and the shared-library plan after backend lease
   consumers land.
3. Run focused tests, full `:website:check`, alternate-port runtime acceptance,
   pull-request CI, merge, and production-safe verification.
4. Record Builder test, update, review, session-memory, and closure artifacts as
   implementation proceeds.
