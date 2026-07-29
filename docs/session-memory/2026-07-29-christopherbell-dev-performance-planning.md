# 2026-07-29 christopherbell.dev Performance Planning

## 10:55 - Approved optimization design and execution-ready plans

### Request

The user asked for a repository-wide review of website performance and
scalability opportunities, including both Java moves into `cbell-lib` and
browser reuse in `static/js/lib`. The user approved the hot-path-first design,
all four design sections, and the written specification. Design/plan approval
does not authorize spoke implementation, publication, merge, or deployment.

### Project Context

- Builder is the workflow hub; `azurras/christopherbell.dev` is the spoke.
- The authoritative checkout at `A:\Projects\christopherbell.dev` was dirty,
  ahead 3, and stale, so it remained untouched.
- An earlier evidence checkout later acquired unrelated documentation changes
  and was preserved without cleanup.
- Final source/range verification used the detached worktree
  `A:\Projects\christopherbell.dev-worktrees\performance-planning-20260729`
  at refreshed `origin/main` commit
  `f31535f29312d24573a6031b0162aa8ebc4b5318`.
- The detached worktree reports only an automatic `gradlew.bat` LF/CRLF
  conversion; no website source was intentionally modified.

### Work Completed

- Updated the approved spec at
  `docs/specs/2026-07-29-christopherbell-dev-performance-scalability-library-optimization.md`.
- Created four literal Code Edit implementation plans:
  - authentication request efficiency;
  - backend query and resource bounds;
  - browser delivery optimization; and
  - shared Java library boundaries.
- Updated the active work record with the approved state, plan links, current
  checkout, validation, blocker, and next execution sequence.
- Refreshed the implementation-plan index.

### Decisions

- Optimize measured hot paths before extracting code.
- Skip authentication for already-public static assets even when a cookie is
  present; use one-read session snapshots with centralized security revocation,
  coalesced conditional activity writes, and CAS-safe rotation/reload behavior.
- Batch conversation unread counts, bound VIN limiter cardinality, add an
  eight-request VIN bulkhead, bound all reviewed upstream bodies, and classify
  every scheduled writer.
- Keep the no-npm/native-module frontend; lazy-load blog, gallery, and media;
  split four feature CSS regions; enforce an 86,434-byte global graph; and hash
  static content instead of Git commits.
- Move stable cursor and generic Mongo lease code to `cbell-lib`, use Gradle
  test fixtures for `TestUtil`, move JJWT dependencies to website, and move the
  single-consumer workflow engine into WFL.
- Keep Bucket4j storage website-owned and feature policy out of shared modules.

### Validation

- All four plans pass
  `.agents/skills/validate-implementation-plan/scripts/validate_implementation_plan.py`.
- Human execution-readiness review found and corrected CAS race behavior,
  nonexistent helper/API references, incomplete security-revocation coverage,
  unbounded Canes response sites, scheduler ownership checks, library test
  dependencies, lazy-loader cleanup behavior, and deterministic Kotlin hashing.
- Placeholder scan found no TBD/TODO/pending-inspection execution gaps.
- `update_hub_indexes.py` refreshed `docs/implementation-plans/index.md`.
- `validate_hub_state.py` passed; it reported only pre-existing legacy-plan
  warnings.
- No website build or runtime test was run because spoke implementation has not
  been authorized and no spoke source was changed.

### Current State

- Builder branch: `main`.
- Hub work remains `active` pending explicit implementation authorization.
- The plans are `ready-for-execution` and ordered so the shared-library plan
  follows the backend plan's new lease consumers.
- No process, database, service, listener, issue, pull request, or production
  state was changed.

### Follow-ups

1. Obtain explicit authorization to execute and the user's choice of
   subagent-driven or inline execution.
2. Refresh `origin/main` and create clean `codex/` implementation worktrees.
3. Execute authentication/backend first, browser independently, then shared
   library boundaries after the backend lease consumers land.
4. For each batch, complete focused/full tests, alternate-port runtime evidence,
   Builder test reports, PR/CI, merge, production-safe verification, and hub
   closeout under the granted authority.
