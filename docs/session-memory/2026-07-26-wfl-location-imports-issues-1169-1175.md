# 2026-07-26 - wfl-location-imports-issues-1169-1175

## 05:28 - WFL and Location Imports Issues 1169-1175

### Request

Continue the approved campaign to complete every open `azurras/christopherbell.dev` issue without routine approval pauses. The user confirmed GitHub authentication and durable authorization to continue. Only GitHub comments from `azurras` were trusted; the batch issues and PR had no trusted scope-changing comments or attachments.

### Project Context

- Builder hub: `C:\Users\Christopher\Developer\builder`, branch `main`.
- Dirty authoritative spoke checkout: `A:\Projects\christopherbell.dev`; it was neither edited nor cleaned.
- Isolated worktree: `A:\Projects\christopherbell.dev-worktrees\wfl-location-imports-1169-1175`, branch `codex/wfl-location-imports-1169-1175`.
- Production is the native Windows `ChristopherBellDev` service on port 8080 with a protected SYSTEM auto-deployment loop.

### Work Completed

- Completed repository-level WFL coordinate/rating queries (#1169), mutually exclusive observable imports (#1170), preview-before-apply (#1171), versioned duplicate cleanup (#1172), public freshness (#1173), typed validated metro/source configuration (#1174), and checksum-idempotent ZIP imports (#1175).
- Manual and scheduled mutation runs share one `MongoLeaseService` lease. `RestaurantImportLeaseGuard` renews at half-duration and verifies ownership before each candidate mutation and after completion.
- Preview tokens are short-lived, one-time, operator-bound, and store only bounded counts/checksum metadata. Apply re-fetches and rejects changed remote snapshots before writes.
- Duplicate cleanup selects the lowest stable ID as survivor and validates every confirmed group version before any deletion.
- PR #1256 passed all checks and squash-merged as `abd2051e76155e5c01137ebec10c2d7550ec3556`; all seven issues closed.

### Decisions

- Existing WFL API contracts remain compatible; additive `2026-07-26` routes expose safer operator workflows and the public freshness DTO.
- MongoDB owns aggregation, durable status, preview identity, TTL cleanup, and atomic lease coordination.
- Public freshness contains source, last successful completion, current/stale state, threshold, and configured city coverage only. Actor details and safe error categories remain protected.
- Production verification avoids destructive authenticated imports and duplicate deletion; it uses focused integration coverage plus public and authorization smoke checks.

### Validation

- Final `:website:check` passed 1,170 Java tests with zero failures or errors and three expected skips; 233 JavaScript tests and sensor-runtime verification passed.
- Packaged final candidate on port 8091 returned 200 for root, WFL pages, and freshness; protected operator APIs returned 403. Disposable databases were dropped and the candidate process stopped.
- Ubuntu CI exposed an existing async MockMvc response race. Waiting for the async result passed four forced focused runs and the complete suite.
- CodeQL identified a lower-case-only HTML regexp in a test assertion. Exact escaped-output/raw-input-absence assertions resolved the alert; all platform, dependency-review, underlying CodeQL, and aggregate CodeQL gates passed.
- Production auto-deployment changed listener PID 6624 to 41176. Local and external root/freshness routes returned 200, protected operator routes returned 403, and MongoDB/website/cloudflared services were Running/Automatic.

### Current State

- Production is serving Batch 6 on PID 41176 after merge `abd2051e`.
- All seven Batch 6 issues are closed and PR #1256 is merged.
- The isolated worktree is clean at pushed head `a861c4b5`; `origin/main` contains the squash merge.
- The campaign has six open issues remaining, exactly #1176-#1181.

### Follow-ups

Refresh the live details for #1176-#1181, create the final isolated worktree and implementation plan from `abd2051e`, and continue the same test-first, review, PR/CI, merge, deployment, and Builder closeout loop without routine approval pauses.
