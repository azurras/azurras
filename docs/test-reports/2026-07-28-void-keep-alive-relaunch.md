# Void Keep Alive Relaunch Test Report

## Document Status

complete

## Story/Issue

Release 1 of the approved Void public-growth program: explain the temporary-post rules, present Likes as Keep alive actions, add sharing, remove the popularity sort, and provide safe social previews and a content-free vanished-post page.

## Branch

- Spoke branch: `codex/void-keep-alive-relaunch`
- Spoke commits: `ba127e8d`, `ba3d2194`
- Candidate base: `7d7d042c26d7bbabee2cdf0bc430127a0020e65e`
- Pull request: `azurras/christopherbell.dev#1314`
- Production merge: `7e958e737b34563d6d49a078243437d5fa9e3377`

## App / Environment

- App: `christopherbell.dev` Spring Boot website
- Candidate profile: `local`
- Candidate port and base URL: `8081`, `http://127.0.0.1:8081`
- Corrected candidate database: isolated Mongo database `christopherbell_void_relaunch_test`
- Mongo endpoint: `mongodb://127.0.0.1:27017`
- Mail: disabled
- Production listener: port 8080, deliberately untouched during candidate testing

## Local Run Details

The corrected candidate was started from `A:\Projects\christopherbell.dev-worktrees\void-keep-alive-relaunch` in a hidden process with these effective settings:

```powershell
$env:GRADLE_USER_HOME='A:\Temp\gradle-void-relaunch'
$env:SPRING_PROFILES_ACTIVE='local'
$env:SERVER_PORT='8081'
$env:SPRING_MONGODB_URI='mongodb://127.0.0.1:27017'
$env:SPRING_MONGODB_DATABASE='christopherbell_void_relaunch_test'
# The local-only JWT default was used and mail was disabled.
.\gradlew.bat :website:bootRun --no-daemon
```

Runtime output was saved under `A:\Temp\void-relaunch-runtime-corrected`. The application process was PID 45164. The exact candidate process tree was stopped after testing, and port 8081 was confirmed closed.

The first candidate invocation supplied only `SPRING_MONGODB_URI`. The local profile separately fixes `spring.mongodb.database` to `christopherbell`, so that invocation wrote one test account, one browser session, and one post to the production database despite the database name in the URI. Production verification exposed the fixture immediately. A read-only cross-collection query identified exactly those three documents; all three were deleted with exact IDs and identifying fields, and follow-up counts confirmed that each was absent. The corrected candidate then supplied both Mongo overrides. Its account and post existed in `christopherbell_void_relaunch_test` and had zero matching documents in `christopherbell`.

## Test Cases

1. Load the anonymous Void feed and confirm the new purpose statement and three lifespan rules.
2. Confirm only Newest and Expiring Soon sorts are available.
3. Create and authenticate an isolated local account, then create a post through the authenticated browser-session path.
4. Keep the post alive and confirm the server extends its expiration by exactly 24 hours.
5. Open the active post page and confirm escaped dynamic social metadata and no-store caching.
6. Open a nonexistent post and confirm a generic, content-free 404 vanished page with no-store caching.
7. Inspect the authenticated active-post controls and accessibility text.
8. Exercise desktop and mobile layouts, including overflow and first-viewport rule visibility.
9. Run the complete repository verification suite.

## Data Sent

- `GET http://127.0.0.1:8081/void`
- `POST http://127.0.0.1:8081/api/accounts/2024-12-15/create` with isolated username `void_corrected_296c8cee` and a local-only email fixture
- `POST http://127.0.0.1:8081/api/accounts/2024-12-15/login` with that fixture account
- `POST http://127.0.0.1:8081/api/posts/2025-09-14/create` with text `Corrected isolated Keep Alive proof`
- `POST http://127.0.0.1:8081/api/posts/2025-09-14/9bdf292c-0abd-448b-b0c2-59418dc53e34/like` in the authenticated browser session
- `GET http://127.0.0.1:8081/p/9bdf292c-0abd-448b-b0c2-59418dc53e34`
- `GET http://127.0.0.1:8081/p/missing-corrected-isolated`
- Browser viewport checks at desktop dimensions and a requested mobile viewport of 390 by 844 CSS pixels
- Cache-busted production `GET` requests to `/`, `/void`, `/messages`, `/music`, `/back-office`, and a nonexistent `/p/{id}`

No passwords, JWT secrets, or session tokens are recorded in this report.

## Response Received

- `/void` status code: 200; the hero states that nothing lasts unless people care, all three lifespan rules are visible, and the Active sort is absent.
- Account-creation status code: 200.
- Login status code: 200.
- Post-creation status code: 201 with post ID `9bdf292c-0abd-448b-b0c2-59418dc53e34`.
- Before Keep alive, the server expiration was July 30, 2026 at 1:52:45 AM UTC.
- Keep alive response: HTTP success with `liked=true` and `extensionHours=24`.
- After Keep alive, the server expiration was July 31, 2026 at 1:52:45 AM UTC, exactly 24 hours later.
- Active-post status code: 200 with `Cache-Control: no-store, max-age=0` and dynamic metadata containing `Corrected isolated Keep Alive proof`.
- Missing-post status code: 404 with a generic vanished message and no post text, post identifier, or post-data API bootstrap.
- Authenticated control: accessible name `Keep alive and add 24 hours`, visible `Keep alive · +24h`, and quiet `1 keep-alives` count.
- Mobile browser: actual client width and scroll width were both 375 pixels, so no horizontal overflow was present; the purpose and rules remained in the first viewport.
- Production `/void`: HTTP 200 at merge SHA `7e958e737b34563d6d49a078243437d5fa9e3377`, with the new hero/rules and no Active sort.
- Production missing post: HTTP 404, generic vanished copy, no requested identifier, and `Cache-Control` containing `no-store`.
- Production `/`, `/messages`, `/music`, and `/back-office`: HTTP 200 at the target merge SHA.
- Production services `ChristopherBellDev`, `ChristopherBellMediaWorker`, `cloudflared`, and `MongoDB`: Running and Automatic.

## Pass / Fail

- Void purpose and rules: PASS.
- Time-only sorting: PASS.
- Keep alive server-authoritative extension: PASS.
- Keep alive labels, count, and accessible name: PASS.
- Safe active-post social preview: PASS.
- Content-free vanished-post response: PASS.
- Desktop and mobile layouts: PASS.
- Production listener isolation: PASS; port 8080 was never changed.
- Corrected database isolation: PASS; the corrected fixture exists only in `christopherbell_void_relaunch_test`.
- Full automated verification: PASS; the final pre-merge `:website:check` completed successfully in 1 minute 34 seconds with all Java, JavaScript, and sensor-runtime checks passing.
- Pull-request CI and security analysis: PASS on Windows, Linux, macOS, dependency review, and all CodeQL jobs.
- Automatic deployment and production smoke: PASS at merge SHA `7e958e737b34563d6d49a078243437d5fa9e3377`.

## Evidence

- Full verification: `$env:GRADLE_USER_HOME='A:\Temp\gradle-void-relaunch'; .\gradlew.bat :website:check --no-daemon` -> `BUILD SUCCESSFUL in 1m 34s`.
- Runtime logs: `A:\Temp\void-relaunch-runtime-corrected`.
- Corrected candidate process: PID 45164, stopped after verification.
- Port check after shutdown: `port_8081_listening=False`.
- Browser DOM checks confirmed the exact visible copy, controls, escaped preview behavior, vanished page, and mobile width measurements described above.
- Static stale-copy scan found no remaining Active sort option or obsolete Like-specific presentation selectors in the changed Void UI.
- Pull request: https://github.com/azurras/christopherbell.dev/pull/1314
- Automatic production release observed at 20:47 America/Chicago on July 28, 2026.
- The deployment listener rotation produced one brief HTTP 502 between healthy responses; the target release then served HTTP 200.
- The unintended production fixture cleanup deleted exactly post `a4f8ef3a-c430-4c1d-b6ce-fc771e2a2cc9`, browser session `3cd85a6c-4747-492a-ace8-7128bb97d0f0`, and account `2e7d40df-5a8d-47dd-be34-3c120cb75e3b`; post-cleanup counts were all zero.

## Bugs / Follow-ups

- No release-blocking product defect remains.
- The first runtime command's database-isolation claim was wrong because the local profile's separate database property won. The fixture was fully removed and the corrected rerun proved isolation. Future local checks must set both `SPRING_MONGODB_URI` and `SPRING_MONGODB_DATABASE`.
- The isolated local Mongo database retains only the corrected test fixture data and is not used by production.
