# Browser Security Issues 1125-1130 Test Report

## Document Status

complete

## Story/Issue

GitHub issues `cbell504/website#1125`, `#1126`, `#1127`, `#1128`, `#1129`, and `#1130`.

## Branch

Spoke branch `codex/browser-security-1125-1130`, rebased onto `origin/main` commit `1de1af0d`; verified head `98099a40`. [PR #1249](https://github.com/azurras/christopherbell.dev/pull/1249) squash-merged as `b6c361d1d916337679a37f04caa46c3475215e71`.

## App / Environment

- App: `christopherbell.dev` Spring Boot application
- Worktree: `A:\Projects\christopherbell.dev-worktrees\browser-security-1125-1130`
- Profile: `local`
- Alternate port and base URL: `8090`, `http://localhost:8090`
- Database: local MongoDB service and the configured `christopherbell` database; runtime checks used anonymous or deliberately unknown account data and did not create or mutate an account
- Relevant environment: `SPRING_PROFILES_ACTIVE=local`, `SERVER_PORT=8090`, `APP_PUBLIC_BASE_URL=http://localhost:8090`, isolated `GRADLE_USER_HOME=A:\Temp\gradle-browser-security-1125-1130`
- Production safety: Windows services `MongoDB`, `ChristopherBellDev`, and `cloudflared` remained running. Pre-merge testing never touched port `8080`; the native auto-deploy later switched production from PID `50708` to PID `26680` after the merge.

## Local Run Details

Started from the isolated spoke worktree with:

```powershell
$env:SPRING_PROFILES_ACTIVE='local'
$env:SERVER_PORT='8090'
$env:APP_PUBLIC_BASE_URL='http://localhost:8090'
$env:GRADLE_USER_HOME='A:\Temp\gradle-browser-security-1125-1130'
.\gradlew.bat :website:bootRun --no-daemon --no-watch-fs --max-workers=1 --console=plain
```

The first Spring child process under test was PID `50408`; the final post-review/rebase process was PID `14672`. Each PID was verified as belonging to this worktree and stopped after its checks. A final socket check showed no listener on `8090`. Production independently advanced from PID `40600` to PID `47220` when a separate mainline update auto-deployed; it remained on `8080`, and `GET http://localhost:8080/` returned `200` after the final cleanup.

## Test Cases

1. Load the anonymous home page and inspect browser security headers and CSRF cookie.
2. Submit logout without a CSRF header.
3. Submit malformed login input with a valid CSRF cookie/header pair.
4. Submit signup input with blank first and last names with a valid CSRF cookie/header pair.
5. Submit a password-reset request with attacker-controlled `X-Forwarded-Host` and `X-Forwarded-Proto` headers.
6. Request the current-account endpoint anonymously.
7. Submit logout with a valid CSRF cookie/header pair and inspect cookie clearing.
8. Run the full Java, focused browser-security Java, JavaScript, syntax, and diff checks supporting the live results.
9. Retest dual login compatibility after review: legacy API mode without CSRF, browser cookie mode without CSRF, and browser cookie mode with CSRF.
10. After merge, inspect the public HTTPS header policy and repeat the login, CSRF, validation, and logout matrix against production.
11. Refresh an already-authenticated Chrome `/shared` session across the deployment boundary and verify fail-closed migration from the removed localStorage JWT session.

## Data Sent

- `GET http://localhost:8090/`
- `GET http://localhost:8090/login` to establish `XSRF-TOKEN`
- `POST http://localhost:8090/api/accounts/2024-12-15/logout` without `X-XSRF-TOKEN`
- `POST http://localhost:8090/api/accounts/2024-12-15/login` with `Content-Type: application/json`, `X-XSRF-TOKEN: <cookie value>`, and body `{"email":"bad","password":""}`
- `POST http://localhost:8090/api/accounts/2024-12-15/create` with the CSRF header and body `{"email":"nobody@example.test","username":"nobody-test","firstName":"","lastName":"","password":"password"}`
- `POST http://localhost:8090/api/accounts/2024-12-15/password-reset/request` with the CSRF header, `X-Forwarded-Host: evil.example`, `X-Forwarded-Proto: https`, and body `{"email":"nobody-browser-security@example.test"}`
- `GET http://localhost:8090/api/accounts/2025-09-03/me` in a fresh anonymous web session
- `POST http://localhost:8090/api/accounts/2024-12-15/logout` with the valid CSRF cookie/header pair
- `POST http://localhost:8090/api/accounts/2024-12-15/login` without the browser-session header or CSRF and malformed body `{"email":"bad","password":""}`
- `POST http://localhost:8090/api/accounts/2024-12-15/login` with `X-CBELL-Browser-Session: cookie`, no CSRF header, and syntactically valid unknown credentials
- The same browser-mode login with the valid CSRF cookie/header pair and malformed body

## Response Received

HTTP response evidence from the running app:

- Anonymous home status code: `200`, response body length `4035`, `XSRF-TOKEN` cookie present.
- `Content-Security-Policy`: `default-src 'self'; base-uri 'self'; object-src 'none'; script-src 'self' https://cdn.jsdelivr.net; style-src 'self' 'unsafe-inline' https://cdn.jsdelivr.net https://maxcdn.bootstrapcdn.com; font-src 'self' data: https://maxcdn.bootstrapcdn.com; img-src 'self' data: blob: https:; connect-src 'self' https://gateway.raisingcanes.com https://order.raisingcanes.com; frame-src https://www.youtube.com https://www.youtube-nocookie.com https://open.spotify.com https://w.soundcloud.com; frame-ancestors 'self'; media-src 'self' blob:; worker-src 'self' blob:; form-action 'self'`.
- `Permissions-Policy`: `camera=(), geolocation=(), microphone=(), payment=(), usb=()`.
- `Referrer-Policy`: `strict-origin-when-cross-origin`; `X-Frame-Options`: `SAMEORIGIN`.
- `Strict-Transport-Security` was absent under the local HTTP profile, as configured.
- Logout without CSRF: `403`.
- Malformed login: `400`.
- Signup with blank names: `400`.
- Spoofed-forwarding password-reset request: generic `200`; the focused controller test separately proves the service receives the configured canonical origin rather than either supplied forwarding header.
- Anonymous current-account request: denied with `403`.
- Logout with CSRF: `200` and two clearing headers: `CBELL_AUTH=... Max-Age=0 ... HttpOnly; SameSite=Lax` and `CBELL_AUTH_STATE=... Max-Age=0 ... SameSite=Lax`.
- Final dual-mode login retest: legacy login without CSRF reached Bean Validation and returned `400`; browser cookie mode without CSRF returned `403`; browser cookie mode with CSRF reached Bean Validation and returned `400`.

Post-merge production evidence from `https://www.christopherbell.dev`:

- Native auto-deploy switched the production listener from PID `50708` to PID `26680`; `/` remained `200` throughout.
- The public response emits the planned CSP, `Strict-Transport-Security: max-age=31536000; includeSubDomains`, `Permissions-Policy: camera=(), geolocation=(), microphone=(), payment=(), usb=()`, `Referrer-Policy: strict-origin-when-cross-origin`, and `X-Frame-Options: SAMEORIGIN`.
- `GET /login` returned `200` and established `XSRF-TOKEN`.
- Malformed legacy API login without CSRF returned `400`; browser cookie-mode login without CSRF returned `403`; the same malformed browser login with a valid token returned `400`.
- `POST /api/accounts/2024-12-15/create` with blank first and last names and a valid CSRF pair returned `400` with both `NotBlank` failures.
- Logout with the valid CSRF pair returned `200` and expired `CBELL_AUTH` as `Secure; HttpOnly; SameSite=Lax` plus `CBELL_AUTH_STATE` as `Secure; SameSite=Lax`.
- Reloading the pre-deployment Chrome `/shared` tab redirected to `/login?redirect=%2Fshared` with no console errors, proving the removed JavaScript-readable JWT is not silently reused after deployment. A fresh cookie login is required by design.

## Pass / Fail

- PASS: browser security response policy is emitted on a live page.
- PASS: unsafe cookie-browser requests require CSRF.
- PASS: malformed authentication/reset inputs and blank signup names are rejected before service work.
- PASS: forwarded-origin spoofing does not change the generic reset workflow; automated boundary verification confirms the configured origin passed to the service.
- PASS: unauthenticated protected data remains denied.
- PASS: logout succeeds with CSRF and expires both browser auth cookies.
- PASS: legacy bearer-token acquisition retains its stateless no-CSRF contract while explicit browser cookie mode remains CSRF protected.
- PASS: alternate-port process cleanup and production continuity checks succeeded.
- PASS: the native Windows auto-deploy completed without downtime and the public HTTPS endpoint exposes the production-only HSTS/cookie policy.
- PASS: a pre-migration browser session fails closed and returns to the intended `/shared` destination after fresh authentication.

## Evidence

- Focused Java command: `.\gradlew.bat :website:test --tests dev.christopherbell.configuration.JwtAuthenticationFilterTest --tests dev.christopherbell.account.AccountControllerTest --tests dev.christopherbell.sharedfolder.SharedFolderSecurityIntegrationTest --no-daemon` — 46 passed.
- Full Java command with isolated cache: `.\gradlew.bat :website:cleanTest :website:test --no-daemon --no-watch-fs --max-workers=1 --console=plain` — final `BUILD SUCCESSFUL`, 108 suites, 999 tests, 0 failures, 3 skipped.
- JavaScript command: `.\gradlew.bat :website:jsTest --no-daemon` — final post-rebase run 195 passed, including the concurrent mainline audio-metadata coverage.
- Focused Node commands for browser auth, signup, shared-folder streaming, and worker runtime — 12 passed.
- `node --check` passed for all 22 JavaScript files changed from the rebased `origin/main`.
- `git diff --check` passed; repository scan found no production `cbellLoginToken`, JavaScript-built bearer header, or token-bearing shared-folder worker path.
- Live runtime interactions were captured at `2026-07-25 20:15:23 -05:00` through PowerShell `Invoke-WebRequest` sessions against port `8090`.
- GitHub checks for PR #1249 passed on Windows, macOS, Ubuntu, Dependency Review, and CodeQL for Actions, Java/Kotlin, and JavaScript/TypeScript.
- Post-merge production interactions were captured from `2026-07-25 20:45:59 -05:00` through `20:47:51 -05:00`, followed by focused public HTTPS requests and Chrome verification.

## Bugs / Follow-ups

- No application defect remained after testing.
- The first broad Gradle retry encountered a missing transient binary result file because another invocation overlapped after a shell timeout. The definitive clean, single-worker run with an isolated Gradle home passed. This was a runner artifact race, not a test assertion failure.
- A successful password login was verified at the controller boundary rather than against the live Mongo database to avoid creating, changing, or exposing production account credentials on the shared host.
- The pre-deployment signed-in Chrome session cannot survive the intentional localStorage-to-HttpOnly-cookie migration. The user must sign in once after deployment; this is expected migration behavior, not a remaining defect.
