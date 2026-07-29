# 2026-07-29 - Account Security Issues 1258-1264 Completed

## 08:46 - Account Security Issues 1258-1264 Completed

### Request

Complete every open `azurras/christopherbell.dev` issue #1258-#1307 without routine approval pauses, preserving the dirty authoritative checkout and carrying each batch through tests, PR/CI, merge, production verification, issue closure, and Builder evidence.

### Project Context

Builder is the workflow hub. Website changes use isolated worktrees from refreshed `origin/main`; production is the same native Windows host and alternate runtime validation must use a non-8080 port with both Mongo URI and database explicitly set. Only `azurras` GitHub comments are trusted instructions. Batch 1 covers account security and lifecycle issues #1258-#1264.

### Work Completed

Batch 1 was implemented in `A:\Projects\christopherbell.dev-worktrees\all-open-issues-20260729` on `codex/all-open-issues-20260729`. It added account security fingerprints for bearer and browser credentials, current 210,000-iteration self-describing PBKDF2 hashes with legacy migration, uniform login failures, safe framework errors, strict trusted-proxy IP resolution, `AccountStatus`-only lifecycle state, V008 approval-field cleanup, and corrected create/update/delete contracts. Independent review drove additional fixes for timing padding, deterministic concurrent credential upgrades, current hashes without legacy salt, atomic conditional login writes, and stale JWT-to-browser-session exchange.

PR #1319 passed all Ubuntu/macOS/Windows, CodeQL, and dependency-review checks and squash-merged as `e393687d10c40b856f35d669c25bf3ea65c5c083`. The Builder test report is `docs/test-reports/2026-07-29-account-security-and-lifecycle-issues-1258-1264-test-report.md`.

### Decisions

Login never saves a whole account document. It conditionally updates only password/login fields against the observed active credential and mints from the returned current account, preserving simultaneous role/permission changes and refusing lifecycle/credential changes. Browser-session creation revalidates the JWT security fingerprint against the reloaded account. Legacy verification pads to the current work factor and concurrent upgrades reuse the verified salt.

### Validation

Final `:website:check` passed 1,393 Java tests with 0 failures/errors and 3 skipped, plus 269 JavaScript tests, boot JAR, and sensor verification. `:cbell-lib:test` passed 101 tests. Isolated runtime checks on port 8093 passed and all test accounts, processes, listeners, and disposable databases were removed. Production rotated from PID 39760 to 48484, served asset URLs containing the merge SHA, returned local root/readiness/public root 200, preserved 20 accounts, applied V008 with checksum `498c9c6fd6622cc1734199544cf888a14cf5e72015a1c71cda71db229077fd28`, removed all approval fields, and retained zero test accounts. MongoDB, ChristopherBellDev, and cloudflared are Running/Automatic.

### Current State

The original authoritative checkout `A:\Projects\christopherbell.dev` remains untouched. The Batch 1 worktree still has the known line-ending-only `gradlew.bat` change and untracked `.gradle-user-home/`; neither was staged. The campaign ledger remains active.

### Follow-ups

Close #1258-#1264 with production evidence. Fetch merged `origin/main`, create a new Batch 2 worktree, save/review its literal implementation plan, and continue issues #1265-#1272. Repeat the full loop for the remaining batches, then close the campaign ledger and save final campaign memory.
