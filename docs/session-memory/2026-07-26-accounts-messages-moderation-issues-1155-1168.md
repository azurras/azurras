# 2026-07-26 - accounts-messages-moderation-issues-1155-1168

## 03:58 - Accounts, Messages, Notifications, Posts, and Moderation Issues 1155-1168

### Request

Continue the approved campaign to complete every open `azurras/christopherbell.dev` issue without routine approval pauses. The user reconfirmed authentication and asked Codex to continue. Only GitHub comments from `azurras` were trusted; the batch issues and PR had no comments or attachments.

### Project Context

- Builder hub: `C:\Users\Christopher\Developer\builder`, branch `main`.
- Dirty authoritative spoke checkout: `A:\Projects\christopherbell.dev`; it was neither edited nor cleaned.
- Isolated worktree: `A:\Projects\christopherbell.dev-worktrees\accounts-messages-moderation-1155-1168`, branch `codex/accounts-messages-moderation-1155-1168`.
- Production is the native Windows `ChristopherBellDev` service on port 8080 with a protected SYSTEM auto-deployment loop.

### Work Completed

- Completed bounded account administration and deletion (#1155-#1156), paged conversations and per-user archive (#1158-#1160), paged/deduplicated/rate-controlled notifications (#1161-#1163), stable post feeds and original-author editing (#1164-#1165), report deduplication and paged moderation (#1166-#1167), and durable redacted moderation audit evidence (#1168).
- Account deletion pseudonymizes retained public/audit identifiers, removes private data and likes, and waits for published media-worker cancellation acknowledgment before deleting artifacts.
- Notification dedupe claims are atomically replaceable after expiry; failed persistence releases both the exact dedupe claim and its exact rate reservation.
- Moderation retry commands preserve the original actor identity and immutable event ID, including when a different administrator retries the operation.
- PR #1255 passed all checks and squash-merged as `5835a3c2b1dc032413e027568583859b9094ab9d`; all 13 issues closed.

### Decisions

- Existing 2025 create/delete API shapes remain compatibility contracts; additive `2026-07-26` routes expose new page and result resources.
- Cursor pagination compares both timestamp and Mongo ID in descending order.
- Public posts survive account deletion only under the stable `deleted-user` identity; private/account-owned records are removed through durable idempotent steps.
- Administrators moderate through the report path but cannot edit another author's post.
- Production verification avoids destructive authenticated mutations and uses local integration coverage plus public and authorization smoke checks.

### Validation

- Final focused review matrix passed 25 tests; independent final re-review found no actionable issues.
- Final `:website:check` passed 1,155 Java tests with zero failures or errors and three expected skips; 231 JavaScript tests passed.
- Packaged final head on port 8090 returned 200 for root, Back Office, and the stable feed; protected admin audit access returned 403. PID 56928 stopped and disposable database `codex_batch5_final_20260726` was removed.
- Ubuntu, macOS, Windows, Dependency Review, and all CodeQL gates passed.
- Production auto-deployment changed listener PID 47288 to 6624. Root, Back Office, and the new stable feed returned 200; protected admin account/audit APIs returned 403.

### Current State

- Production is serving Batch 5 on PID 6624 after merge `5835a3c2`.
- All 13 Batch 5 issues are closed and PR #1255 is merged.
- The isolated worktree is clean at pushed head `349ba1c7`; `origin/main` is the squash merge.
- The campaign has 13 open issues remaining, exactly #1169-#1181.

### Follow-ups

Refresh the live issue inventory, group #1169-#1181 into the final coherent batch or batches, create a clean worktree from current `origin/main`, and continue the same spec, plan, test, review, PR/CI, merge, deployment, and Builder closeout loop without routine approval pauses.
