# 2026-08-04 christopherbell.dev Modular Monolith Plan

## 20:43 - Approved design and executable foundation plan

### Request

The user asked for a plan to turn `christopherbell.dev` into a modular monolith. They confirmed the target was the registered `christopherbell.dev` spoke, selected boundary safety over build isolation or future service extraction, selected incremental slices over a coordinated rewrite, kept frontend assets outside module enforcement, chose Spring Modulith plus ArchUnit, approved each design section, and approved the written specification.

### Project Context

- Builder is the workflow hub at `C:\Users\Christopher\Developer\builder`; the authoritative spoke is `A:\Projects\christopherbell.dev` with canonical remote `https://github.com/azurras/christopherbell.dev.git`.
- The authoritative spoke checkout is intentionally dirty and stale: it was three commits ahead and 120 commits behind `origin/main` during planning. It was not modified.
- Local `origin/main` matched remote `main` at `9c587103cb7f7df2ab52ed3e232f1ca67660fd6e` on 2026-08-04.
- The current application contains 626 Java files in 18 top-level website packages. A source-import inventory found 449 cross-package imports across 71 dependency directions before excluding library ownership and normalizing `permission` to `account`.
- The repository builds one Spring Boot 4.1.0/Java 25 `website` deployable plus `cbell-lib`, uses Gradle dependency verification, and runs the root `build` task on Linux, macOS, and Windows CI.

### Work Completed

- Created and pushed the approved specification at `docs/specs/2026-08-04-christopherbell-dev-modular-monolith.md`; its status is now `ready-for-execution`.
- Created `docs/implementation-plans/2026-08-04-christopherbell-dev-modular-monolith-foundation.md` as the first ready-for-execution plan.
- Decomposed the full migration into a foundation delivery plus separate inspected plans for account/authorization, bootstrap/configuration, lower-coupled capabilities, social capabilities, host-heavy capabilities, and final graph closure.
- The foundation plan contains four independently reviewable tasks:
  1. Add Spring Modulith 2.1.0 test-only dependencies and explicitly annotated discovery.
  2. Implement normalized ArchUnit dependency rules with API/internal/orchestration fixtures.
  3. Generate and freeze a read-only legacy violation store.
  4. Generate module documentation, document contributor commands, run full checks, inspect the boot JAR, and verify a packaged candidate on an alternate port.
- Added literal Code Edit blocks with current/proposed code, exact line ranges, task-level verification, Before-Edit Briefs, TDD steps, commits, rollback, risks, and completion criteria.
- Refined the plan during review to forward only the two approved `archunit.*` store flags into Gradle test workers, reject uncatalogued top-level production packages, normalize violations without source line numbers, and own candidate process/database cleanup precisely.

### Decisions

- Keep one boot JAR and do not create Gradle projects per capability.
- Use Spring Modulith 2.1.0 for test-time discovery, verification, focused tests, and documentation only. Do not add runtime Modulith features, events persistence, actuator exposure, or an outbox.
- Use explicitly annotated discovery so packages opt into closed-module enforcement one migration at a time.
- Keep a version-controlled ArchUnit frozen store as a monotonic ratchet. Creation and updates are disabled by default; explicit update commands must produce reviewed removals only.
- Treat `permission` as account ownership in the baseline and make `libs` the only external top-level area permitted outside the website catalog.
- Permit cross-area access only through a target `.api` package, while separately forbidding business dependencies on `admin`, `configuration`, and `view` even if API-shaped.
- Preserve frontend organization, HTTP contracts, MongoDB schemas, security behavior, Windows deployment, and production port 8080 throughout boundary extraction.

### Validation

- `validate_implementation_plan.py` passed after correcting every mechanical error.
- Placeholder/red-flag scans returned no matches.
- Literal new-file line ranges were compared with proposed code block line counts and match.
- Human execution review found no blockers after adding ArchUnit property forwarding, unknown-package catalog enforcement, and exact runtime ownership/cleanup.
- `update_hub_indexes.py` refreshed the implementation-plan index.
- `validate_hub_state.py` passed. It reported only pre-existing warnings for legacy July implementation plans without modern Code Edit blocks.
- `git diff --check` passed.
- No spoke code, Gradle command, Java test, application process, MongoDB database, service, or production listener was changed or run; this request produced planning artifacts only.

### Current State

- Builder remains on `main` with only the new foundation plan, approved spec status change, generated indexes, and this session-memory file pending the final checkpoint commit.
- The authoritative spoke checkout remains untouched.
- No helper process or local application is running from this work.

### Follow-Ups

1. Choose execution mode: subagent-driven task execution with review gates or inline execution with checkpoints.
2. At execution, invoke `superpowers:using-git-worktrees`, refresh `origin/main`, create `codex/modular-monolith-foundation`, and revalidate plan line ranges if the base SHA has moved.
3. Invoke `write-jane-street-style-code` before every code-changing task.
4. Complete the foundation through focused tests, full checks, alternate-port verification, PR/CI, merge, protected deployment, production checks, Builder test report, and closeout.
5. Create the account/authorization line-level plan only after the foundation is merged and its actual baseline is available.
