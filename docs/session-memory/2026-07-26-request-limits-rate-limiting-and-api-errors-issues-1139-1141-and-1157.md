# 2026-07-26 - request-limits-rate-limiting-and-api-errors-issues-1139-1141-and-1157

## 00:52 - Request Limits, Rate Limiting, and API Errors Issues 1139-1141 and 1157

### Request

Continue the approved campaign to complete every open `azurras/christopherbell.dev` issue without routine approval pauses. The user explicitly approved continued execution and asked to resume after login. Only GitHub comments from `azurras` were trusted; the batch issues and PR had no comments or attachments.

### Project Context

- Builder hub: `C:\Users\Christopher\Developer\builder`, branch `main`.
- Dirty authoritative spoke checkout: `A:\Projects\christopherbell.dev`; it was neither edited nor cleaned.
- Isolated worktree: `A:\Projects\christopherbell.dev-worktrees\request-limits-api-errors-1139-1141-1157`, branch `codex/request-limits-api-errors-1139-1141-1157`.
- Production is the native Windows `ChristopherBellDev` service on port 8080 with a protected SYSTEM auto-deployment loop.

### Work Completed

- Completed configurable route-aware request limits (#1139), bounded inactive rate-bucket expiry (#1140), standard rate response headers and envelope (#1141), and explicit safe service failure exceptions (#1157).
- Initial commit `d70e05d96f4c24b897f082368f9d268e804d6452` implemented typed request limits, native-Jackson standard boundary responses, bounded rate state, consumption metadata, and ten exact typed service-failure translations.
- Independent review found three important boundary concerns. Commit `fb1f1c557bc19b02adb00e4dce8c8d9b1de9b390` replaced linear expiry scanning with an ordered expiry index, preserved streamed shared-upload chunks, rejected duplicate rule names, isolated bucket identity by rule index, and added focused regressions.
- PR #1254 passed all checks and squash-merged as `ac74bbe30e7392781950bbc1f06f44e196adc46e`; issues #1139, #1140, #1141, and #1157 closed automatically.

### Decisions

- Ordinary unknown-length bodies pre-read only limit-plus-one and replay accepted bytes so parsers cannot stop before overflow is observed; feature-owned shared-upload chunks retain streaming enforcement.
- Rate buckets use synchronized access-ordered hard bounds plus an expiry-ordered index, avoiding an all-client scan on every request while retaining sliding inactivity expiry.
- Rule names are case-insensitively unique, and internal bucket identity also includes the rule index as defense in depth.
- Only identified persistence/password-provider operational wrappers became typed exceptions; programmer faults continue to the existing generic handler.

### Validation

- Witnessed compile RED, ten exact service RED failures, unknown-length early-reader RED, duplicate-rule RED, and upload-streaming RED.
- Final focused review-fix matrix passed 23 tests.
- Final `cleanTest check` passed 1,173 Java tests with zero failures and three expected skips; `bootJar`, JavaScript checks, and sensor runtime verification passed.
- Packaged final head on port 8090 returned root 200, standard 413 for known-length and raw chunked oversize bodies, 400 for the first small login, and 429 with `Retry-After`, limit, remaining, and reset headers for the second. PID 50588 stopped and exact disposable database `christopherbell_request_limits_final_20260726003951` was dropped.
- Ubuntu, macOS, Windows, Dependency Review, and all CodeQL gates passed.
- Production auto-deployment changed listener PID 20156 to 47288; `/` stayed 200 and readiness settled from 503 to 200.

### Current State

- Production is healthy on PID 47288 after merge `ac74bbe3`.
- The four issues are closed and PR #1254 is merged.
- The isolated worktree is clean at pushed head `fb1f1c55`; `origin/main` is the squash merge.
- The campaign has 26 open issues remaining.

### Follow-ups

Refresh Builder indexes and validation, commit/push this closeout checkpoint, then reconcile the live open-issue inventory and start the next dependency-aware batch without requesting routine approval.
