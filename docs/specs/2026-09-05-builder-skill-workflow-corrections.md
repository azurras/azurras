# Builder Skill Workflow Corrections

## Document Status
complete

## Purpose
Implement the user's selected corrections from the skill audit: guarded Builder commits, consistent delivery closure, maintainable implementation plans, safe Spring verification/deployment routing, and Windows spoke registration.

## Requirements
- Commit only explicitly selected repository-relative files. Preserve unrelated working and staged changes by refusing an unrelated staged index; support explicit push-only recovery after commit success and push failure.
- Record required continuity evidence before issue closure. Runtime evidence and a test report are required when application runtime changes or runtime verification was requested; document a reason when not applicable.
- Accept inspected file/symbol implementation contracts with dependencies, behavior, invariants, boundaries, effects, and concrete verification. Literal patches remain optional and supported for existing plans.
- Separate alternate-port verification from authorized deployment. Verify effective test database isolation before starting the candidate, use the existing service/deployment mechanism, and establish rollback and health criteria before any production mutation.
- Discover the actual Builder root on Windows or macOS; show PowerShell-native registration commands.
- Update affected skill metadata, repository policy, status guidance, and behavioral tests together. Do not modify unselected Superpowers skills or consolidate unrelated skills.

## Validation Plan
Run real Git scenarios against disposable repositories and local bare remotes, plan parser/save CLI tests, existing Builder tests, independent instruction scenarios, skill metadata checks, index checks, and hub validation. No live application restart or production database access is needed.

## Open Questions
None. The user's selection authorizes these corrections. Builder changes use its scoped main-branch commit/push workflow.
