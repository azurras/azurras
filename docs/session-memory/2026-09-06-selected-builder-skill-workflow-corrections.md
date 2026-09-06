# 2026-09-06 - Selected Builder skill workflow corrections

## 08:14 - Selected Builder skill workflow corrections

### Request
Update the five selected workflows from the skill audit: commit-push-builder-main, complete-story-issue, save-implementation-plan and its review/validation skills, verify-local-spring-app, and register-spoke-repo. The user subsequently said Continue. Related closure/hub validation, metadata, and repository policy changes are included to keep those contracts consistent.

### Work Completed
- Commit helper now requires exact repeated --path selections or explicit --push-only. It refuses unrelated staged files, treats filenames literally, validates tracked deletions exactly, rejects broad/deleted directories before staging, and preserves the index during push-only recovery. It never force-pushes.
- Delivery uses one Runtime Evidence Required classification. App runtime changes or requested app checks require runtime proof and a real report; documentation and standalone tooling use native evidence with a recorded reason. Continuity precedes closure; actual closure results follow readback. Parked work gets an honest incomplete snapshot/status update and stays open.
- New plans use Plan Format task-contract-v1: inspected files/symbols, per-task contracts, dependencies, and verification; literal replacement code is optional. Unversioned historical literal plans preserve their old whole-plan checks. Save/review/validate and hub routing agree. Only eight explicitly named pre-schema historical plans remain warning-only.
- Spring local verification checks effective test database isolation before database-backed tests/startup, uses an alternate port, and cleans up the candidate. Previously authorized production deployment uses the repository service/deployment mechanism, recovery procedure, and application health checks. Testing alone does not trigger production restart.
- Registration discovers/verifies the active Windows or macOS Builder root and supplies --root explicitly; its example is PowerShell-native.
- Updated nine skill entrypoints and metadata, AGENTS.md, status guidance, helper behavior, and regression tests. Unselected Superpowers skills were not changed.

### Verification Evidence
- python -B -m unittest discover -s .agents/tests: 46 passed (7.610 seconds in final run).
- Git integration scenarios use disposable repositories and local bare remotes. Inputs include selected literal filenames, unrelated staged changes, invalid paths, a deleted directory, tracked deletion, and a remote configured to reject a push. Assertions verify exact committed paths, unchanged index on refusal/dry-run, and successful --push-only recovery with HEAD/index preserved.
- Plan regressions witnessed rejection of the supported inspected-contract input before implementation; save CLI now writes valid contracts and refuses invalid ones before file creation. Negative cases cover missing/empty task fields, unresolved inspection, missing tasks, empty status, versioned literal plans with empty Branch/Risks/Rollback, and malformed new plans through hub validation. Legacy non-edit delivery tasks remain compatible.
- Nine affected skills passed skill-creator quick_validate.py. All 22 agents/openai.yaml files parsed with PyYAML.
- Hub indexes current; validate-hub-state passed with the same eight historical-plan warnings. git diff --check passed.
- Independent agent evaluated docs-only closure, inspected-symbol plans, Windows Spring verification, Windows registration, missing required runtime proof, already-authorized managed deployment, and parked work. Review found deleted-directory selection and versioned empty-section defects; both reproduced, fixed, and covered by passing tests. Final independent response: no remaining review blockers.

### Delivery
Spec checkpoint e9f8a06 and plan checkpoint 071ce00 were pushed before implementation. Implementation commit 9fbf6ec was pushed to origin/main using the updated helper and the exact 26 reviewed files after a dry-run preview.

Source issue/PR closure is not applicable: this is a direct Builder task, and Builder policy explicitly permits its scoped main-branch workflow. App runtime testing/report and live deployment are not applicable; actual Builder CLI/Git behavior was exercised. No Spring service or production database was touched.

### Decisions and Limits
The approved contract-plan format was documented before its validator implementation; its initial structural rejection served as baseline evidence. Historical plans are preserved rather than rewritten to satisfy a new format. Structural validation does not establish the truth of claimed inspection or semantic correctness; readiness review remains required.

### Current State and Follow-ups
Implementation is published. Spec/plan are marked complete; this continuity record and generated indexes are the final publication checkpoint. No functional follow-ups remain within the five selected updates. The eight historical warnings are pre-existing compatibility exceptions.
