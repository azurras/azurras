# 2026-08-21 PostgreSQL Shadow Rehearsal Merged

## 11:18 - Task 8 delivery and PR integration complete

### Request

Continue the approved christopherbell.dev PostgreSQL migration autonomously until the project is finished. Preserve the dirty authoritative checkout, keep production untouched during rehearsal, commit and push completed branch work, and stop only for a genuinely new authority requirement.

### Project Context

- Builder hub: `C:\Users\Christopher\Developer\builder`, branch `main`.
- Spoke: `azurras/christopherbell.dev`.
- Isolated worktree: `A:\Projects\christopherbell.dev-worktrees\postgresql-migration`, branch `codex/postgresql-migration`.
- Authoritative checkout: `A:\Projects\christopherbell.dev`; its unrelated `gradlew.bat` change remained untouched.
- Production remained the website on 8080, MongoDB on 27017, and PostgreSQL 16 on 5432. Task 8 did not mutate those services or production data.

### Work Completed

- Implemented, independently reviewed, and merged Tasks 1 through 8 of the PostgreSQL migration plan.
- Repeated all-52-kind SHADOW and RECONCILE operations against the read-only live Mongo source and a 63,230-document restored production archive.
- Verified the PostgreSQL-only candidate on port 18087 with separated application and bridge roles, 39 exact HTTP checks, scheduler/capacity/query-plan evidence, secret scans, and exact cleanup.
- Opened and merged [PR #1370](https://github.com/azurras/christopherbell.dev/pull/1370). Final branch head `ced5b7cb1f1c9feb3c4e973a09fea7058ffbb497` was squash-merged as `bca4231b4d36bdad963a4d33645b5bb61d88795c`; the trees match exactly.
- Corrected CI so PostgreSQL 18 generates exact-revision jOOQ sources once for the OS matrix and independently for Java CodeQL. Generated sources remain uncommitted.
- Set the website Gradle test worker to its proven 2 GiB heap after all three CI runners exhausted the default 512 MiB worker.
- Corrected the POSIX authority-test helper to protect files as `0600` and directories as `0700`; the prior helper removed owner directory traversal and caused `AccessDeniedException` on macOS/Linux.
- Added the merged spoke update and final review, and refreshed the active work record.

### Decisions

- Used a squash merge to match the repository's existing PR history convention.
- Kept Task 8 strictly non-authoritative: no FINALIZE command, production listener rotation, service dependency mutation, or authority marker.
- Treated CI failures as product evidence: fixed exact missing jOOQ generation, heap capacity, and POSIX test portability rather than bypassing or skipping checks.
- Kept the overall Builder work `active`; Task 8 is complete, while Tasks 9 and 10 remain.

### Validation

- Definitive local gate: 2,386 Java tests, 2,311 passed, 75 expected skips, zero failures/errors.
- Candidate matrix: 39/39 exact expected statuses and 39/39 below 2,000 ms; maximum 75.298 ms.
- Live and archive migration evidence: all 52 kinds reconciled repeatedly; archive contained 63,230 documents.
- Final PR checks all passed: jOOQ code generation, macOS, Ubuntu, Windows, Java/JavaScript/Actions CodeQL, and dependency review.
- Merge readback: PR state `MERGED`, merge commit `bca4231b`, final branch/main tree equivalence true.
- Disposable candidates, databases, roles, PIDs, listeners, secrets, and scratch paths were cleaned; production remained unchanged.

### Current State and Follow-ups

- Task 8 is complete and on `origin/main`.
- Task 9 is a production authority cutover. It requires an explicitly approved maintenance window before stopping writers, taking the final backup, running FINALIZE, publishing the one-way authority marker, or rotating the live listener.
- After Task 9, retain Mongo stopped and frozen while collecting 14 full days of PostgreSQL soak and restore evidence. Task 10 then removes Mongo runtime code/service/data under its retention gates and closes the initiative.
