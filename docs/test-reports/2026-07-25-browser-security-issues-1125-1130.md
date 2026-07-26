# Browser Security Issues 1125-1130 Test Report

## Document Status

complete

## Story/Issue

GitHub issues `cbell504/website#1125`, `#1126`, `#1127`, `#1128`, `#1129`, and `#1130`.

## Branch

Spoke branch `codex/browser-security-1125-1130`, rebased onto `origin/main` commit `1de1af0d`; verified head `98099a40`.

## App / Environment

- App: `christopherbell.dev` Spring Boot application
- Worktree: `A:\Projects\christopherbell.dev-worktrees\browser-security-1125-1130`
- Profile: `local`
- Alternate port and base URL: `8090`, `http://localhost:8090`
- Database: local MongoDB service and the configured `christopherbell` database; runtime checks used anonymous or deliberately unknown account data and did not create or mutate an account
- Relevant environment: `SPRING_PROFILES_ACTIVE=local`, `SERVER_PORT=8090`, `APP_PUBLIC_BASE_URL=http://localhost:8090`, isolated `GRADLE_USER_HOME=A:\Temp\gradle-browser-security-1125-1130`
- Production safety: Windows services `MongoDB`, `ChristopherBellDev`, and `cloudflared` remained running. Production stayed on PID `40600`, port `8080`.

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

## Pass / Fail

- PASS: browser security response policy is emitted on a live page.
- PASS: unsafe cookie-browser requests require CSRF.
- PASS: malformed authentication/reset inputs and blank signup names are rejected before service work.
- PASS: forwarded-origin spoofing does not change the generic reset workflow; automated boundary verification confirms the configured origin passed to the service.
- PASS: unauthenticated protected data remains denied.
- PASS: logout succeeds with CSRF and expires both browser auth cookies.
- PASS: legacy bearer-token acquisition retains its stateless no-CSRF contract while explicit browser cookie mode remains CSRF protected.
- PASS: alternate-port process cleanup and production continuity checks succeeded.

## Evidence

- Focused Java command: `.\gradlew.bat :website:test --tests dev.christopherbell.configuration.JwtAuthenticationFilterTest --tests dev.christopherbell.account.AccountControllerTest --tests dev.christopherbell.sharedfolder.SharedFolderSecurityIntegrationTest --no-daemon` — 46 passed.
- Full Java command with isolated cache: `.\gradlew.bat :website:cleanTest :website:test --no-daemon --no-watch-fs --max-workers=1 --console=plain` — final `BUILD SUCCESSFUL`, 108 suites, 999 tests, 0 failures, 3 skipped.
- JavaScript command: `.\gradlew.bat :website:jsTest --no-daemon` — final post-rebase run 195 passed, including the concurrent mainline audio-metadata coverage.
- Focused Node commands for browser auth, signup, shared-folder streaming, and worker runtime — 12 passed.
- `node --check` passed for all 22 JavaScript files changed from the rebased `origin/main`.
- `git diff --check` passed; repository scan found no production `cbellLoginToken`, JavaScript-built bearer header, or token-bearing shared-folder worker path.
- Live runtime interactions were captured at `2026-07-25 20:15:23 -05:00` through PowerShell `Invoke-WebRequest` sessions against port `8090`.

## Bugs / Follow-ups

- No application defect remained after testing.
- The first broad Gradle retry encountered a missing transient binary result file because another invocation overlapped after a shell timeout. The definitive clean, single-worker run with an isolated Gradle home passed. This was a runner artifact race, not a test assertion failure.
- A successful password login was verified at the controller boundary rather than against the live Mongo database to avoid creating, changing, or exposing production account credentials on the shared host.
