# Void Keep Alive Relaunch Test Report

## Document Status

complete

## Story/Issue

Release 1 of the approved Void public-growth program: explain the temporary-post rules, present Likes as Keep alive actions, add sharing, remove the popularity sort, and provide safe social previews and a content-free vanished-post page.

## Branch

- Spoke branch: `codex/void-keep-alive-relaunch`
- Spoke commits: `ba127e8d`, `ba3d2194`
- Candidate base: `7d7d042c26d7bbabee2cdf0bc430127a0020e65e`
- Pull request and production merge: pending at the time of this candidate report

## App / Environment

- App: `christopherbell.dev` Spring Boot website
- Candidate profile: `local`
- Candidate port and base URL: `8081`, `http://127.0.0.1:8081`
- Candidate database: isolated Mongo database `christopherbell_void_relaunch_test`
- Mongo endpoint: `mongodb://127.0.0.1:27017`
- Mail: disabled
- Production listener: port 8080, deliberately untouched during candidate testing

## Local Run Details

The candidate was started from `A:\Projects\christopherbell.dev-worktrees\void-keep-alive-relaunch` in a hidden process with these effective settings:

```powershell
$env:GRADLE_USER_HOME='A:\Temp\gradle-void-relaunch'
$env:SPRING_PROFILES_ACTIVE='local'
$env:SERVER_PORT='8081'
$env:SPRING_MONGODB_URI='mongodb://127.0.0.1:27017/christopherbell_void_relaunch_test'
# A local-only JWT secret was supplied and mail was disabled.
.\gradlew.bat :website:bootRun --no-daemon
```

Runtime output was saved under `A:\Temp\void-relaunch-runtime`. The application process was PID 26752 and reported startup in 4.413 seconds. The exact candidate process was stopped after testing, and port 8081 was confirmed closed. Gradle consequently reported its long-running `bootRun` task as interrupted after the intentional stop; application startup and all runtime checks had already passed.

## Test Cases

1. Load the anonymous Void feed and confirm the new purpose statement and three lifespan rules.
2. Confirm only Newest and Expiring Soon sorts are available.
3. Create and authenticate an isolated local account, then create a post containing metadata-sensitive characters.
4. Keep the post alive and confirm the server extends its expiration by exactly 24 hours.
5. Open the active post page and confirm escaped dynamic social metadata and no-store caching.
6. Open a nonexistent post and confirm a generic, content-free 404 vanished page with no-store caching.
7. Inspect the authenticated active-post controls and accessibility text.
8. Exercise desktop and mobile layouts, including overflow and first-viewport rule visibility.
9. Run the complete repository verification suite.

## Data Sent

- `GET http://127.0.0.1:8081/void`
- `POST http://127.0.0.1:8081/api/accounts/2024-12-15/create` with an isolated local username and email fixture
- `POST http://127.0.0.1:8081/api/accounts/2024-12-15/login` with that fixture account
- `POST http://127.0.0.1:8081/api/posts/2025-09-14/create` with text `Runtime <metadata> proof for the Void`
- `POST http://127.0.0.1:8081/api/posts/2025-09-14/a4f8ef3a-c430-4c1d-b6ce-fc771e2a2cc9/like` in the authenticated browser session
- `GET http://127.0.0.1:8081/p/a4f8ef3a-c430-4c1d-b6ce-fc771e2a2cc9`
- `GET http://127.0.0.1:8081/p/missing-runtime-post`
- Browser viewport checks at desktop dimensions and a requested mobile viewport of 390 by 844 CSS pixels

No passwords, JWT secrets, or session tokens are recorded in this report.

## Response Received

- `/void` status code: 200; the hero states that nothing lasts unless people care, all three lifespan rules are visible, and the Active sort is absent.
- Account-creation status code: 200.
- Login status code: 200.
- Post-creation status code: 201 with post ID `a4f8ef3a-c430-4c1d-b6ce-fc771e2a2cc9`.
- Before Keep alive, the server expiration was July 30, 2026 at 1:29:53 AM.
- Keep alive response: HTTP success with `liked=true` and `extensionHours=24`.
- After Keep alive, the server expiration was July 31, 2026 at 1:29:53 AM, exactly 24 hours later.
- Active-post status code: 200 with `Cache-Control: no-store, max-age=0`; metadata contains escaped `Runtime &lt;metadata&gt; proof` rather than executable markup.
- Missing-post status code: 404 with a generic vanished message and no post text, post identifier, or post-data API bootstrap.
- Authenticated control: accessible name `Keep alive and add 24 hours`, visible `Keep alive · +24h`, and quiet `1 keep-alives` count.
- Mobile browser: actual client width and scroll width were both 375 pixels, so no horizontal overflow was present; the purpose and rules remained in the first viewport.

## Pass / Fail

- Void purpose and rules: PASS.
- Time-only sorting: PASS.
- Keep alive server-authoritative extension: PASS.
- Keep alive labels, count, and accessible name: PASS.
- Safe active-post social preview: PASS.
- Content-free vanished-post response: PASS.
- Desktop and mobile layouts: PASS.
- Production listener isolation: PASS; port 8080 was never changed.
- Full automated verification: PASS; `:website:check` completed successfully in 1 minute 30 seconds with all Java, JavaScript, and sensor-runtime checks passing.

## Evidence

- Full verification: `$env:GRADLE_USER_HOME='A:\Temp\gradle-void-relaunch'; .\gradlew.bat :website:check --no-daemon` -> `BUILD SUCCESSFUL in 1m 30s`.
- Runtime logs: `A:\Temp\void-relaunch-runtime`.
- Candidate process: PID 26752, stopped after verification.
- Port check after shutdown: `port_8081_listening=False`.
- Browser DOM checks confirmed the exact visible copy, controls, escaped preview behavior, vanished page, and mobile width measurements described above.
- Static stale-copy scan found no remaining Active sort option or obsolete Like-specific presentation selectors in the changed Void UI.

## Bugs / Follow-ups

- No release-blocking defect was found.
- Pull-request CI, automatic deployment, and production smoke checks remain delivery steps after this candidate report; their outcomes will be added to the final session record.
- The isolated local Mongo database retains only the test fixture data and is not used by production.
