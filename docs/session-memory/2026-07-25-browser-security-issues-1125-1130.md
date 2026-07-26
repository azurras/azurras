# 2026-07-25 Browser Security Issues 1125-1130

## 20:51 - Complete browser security sub-batch

### Request

Continue the autonomous campaign to complete every open `azurras/christopherbell.dev` GitHub issue.
The user authorized routine continuation without approval pauses and asked that this preference be
saved. Only comments by `azurras` may influence issue scope or acceptance.

### Project Context

Builder is the durable workflow hub at `C:\Users\Christopher\Developer\builder`. The authoritative
spoke checkout at `A:\Projects\christopherbell.dev` contains extensive unrelated user work and was
left untouched. Development used the isolated worktree
`A:\Projects\christopherbell.dev-worktrees\browser-security-1125-1130` on
`codex/browser-security-1125-1130`. The development host also runs production on native Windows
services, so all pre-merge runtime testing used port `8090`.

### Work Completed

- Delivered issues #1125-#1130 in [PR #1249](https://github.com/azurras/christopherbell.dev/pull/1249), squash-merged as `b6c361d1d916337679a37f04caa46c3475215e71`.
- Added typed browser-security configuration, production HSTS, CSP, same-origin framing, referrer and permissions policies.
- Restored Spring SPA CSRF for cookie browsers while retaining a narrow stateless legacy/API login path.
- Replaced browser localStorage JWT handling with an HttpOnly `CBELL_AUTH` cookie and a non-secret `CBELL_AUTH_STATE` marker verified against `/me`.
- Removed token transport from browser modules and the shared-folder service worker; same-origin requests now carry cookies naturally.
- Made password-reset links use the configured canonical origin and added bounded Bean Validation to login/reset DTOs.
- Required and normalized first and last names in signup UI and server validation.
- Recorded the final review at `docs/spoke-reviews/2026-07-25-browser-security-issues-1125-1130.md` and production evidence in `docs/test-reports/2026-07-25-browser-security-issues-1125-1130.md`.

### Decisions

- Explicit bearer clients retain priority and receive the legacy JWT response; only requests opting into `X-CBELL-Browser-Session: cookie` receive browser cookies and require CSRF for login.
- A readable cookie contains only a presence marker, never credentials or authorization claims; the server remains authoritative.
- Invalid or expired public credentials continue anonymously so browser logout and stale-session cleanup can complete.
- Production trusts the configured canonical reset origin and ignores forwarded host/protocol values.

### Validation

- Observed RED evidence before implementation for cookie auth, CSRF, localStorage removal, signup names, and worker cookie forwarding.
- Focused post-review Java: 74 cases passed.
- Full Java: 108 suites, 999 tests, 0 failures, 3 skipped.
- Full JavaScript after rebase: 195/195 passed; `node --check` passed all 22 changed JavaScript files; `git diff --check` passed.
- PR #1249 passed Windows, macOS, Ubuntu, Dependency Review, and CodeQL for Actions, Java/Kotlin, and JavaScript/TypeScript.
- Live port-8090 matrix passed, the process was stopped, and production port 8080 stayed healthy.
- Native auto-deploy switched production from PID `50708` to PID `26680`; the public site remained `200` and now emits the complete HTTPS security policy.
- Production login/CSRF/validation/logout probes returned the planned `400`/`403`/`200` results and Secure cookie-clearing attributes.
- A pre-deployment signed-in Chrome `/shared` session redirected to `/login?redirect=%2Fshared` after refresh, proving fail-closed migration from the removed localStorage JWT. No console errors appeared.

### Current State

- Issues #1125-#1130 are closed automatically by the merged PR.
- The isolated worktree is clean at `98099a40`; `origin/main` is `b6c361d1d916337679a37f04caa46c3475215e71`.
- Production is healthy on port `8080`, PID `26680`.
- Chrome is waiting at the login page with the intended `/shared` return target because the storage migration requires one fresh login.
- The campaign has 41 open issues remaining.

### Follow-ups

- After the user completes the one-time fresh login, confirm authenticated `/shared` access under the new cookie session.
- Select the next dependency-aware issue batch and continue the full Builder delivery loop.
