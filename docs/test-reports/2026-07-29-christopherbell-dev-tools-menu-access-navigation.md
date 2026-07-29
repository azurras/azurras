# ChristopherBell.dev Tools Menu Access Navigation Test Report

## Document Status

complete

## Story/Issue

Move Music, Command Center, and Back Office into the Tools menu, show each destination only to accounts with effective access, and sort the visible Tools list alphabetically.

## Branch

`codex/tools-menu-access-navigation`, based on refreshed `origin/main` commit `e393687d10c40b856f35d669c25bf3ea65c5c083`.

## App / Environment

- App: `christopherbell.dev` Spring Boot website
- Profile: `local`
- Listener: `127.0.0.1:8092`
- Base URL: `http://127.0.0.1:8092`
- Database: isolated MongoDB database `christopherbell_tools_menu_20260729`
- Relevant environment: `SPRING_PROFILES_ACTIVE=local`, `SERVER_PORT=8092`, and the isolated MongoDB database name
- Production listener: port 8080 remained untouched

## Local Run Details

The packaged application was started from the isolated spoke worktree with `java -jar website\build\libs\website.jar` after setting the local profile, port 8092, and isolated database environment. Startup and request logs were observed in the foreground process output; no separate log file was written. The Java child process was PID 10476. After testing, that exact process was stopped, port 8092 was confirmed closed, and the isolated database was dropped successfully with result `{"dropped":1}`.

## Test Cases

1. Open the local home page as a signed-out visitor and expand Tools.
2. Confirm Music is absent from the top-level navigation.
3. Confirm Back Office, Command Center, Music, and Shared Folder are absent from signed-out Tools.
4. Confirm the remaining public Tools entries are alphabetized.
5. Exercise the pure account-access projection for anonymous, ordinary user, Music reader, Music writer, Shared Folder reader, and administrator account shapes.
6. Exercise the final menu builder for each access projection and confirm protected destinations appear only with effective access.
7. Run the complete JavaScript and Gradle verification suites.

## Data Sent

- Browser navigation: `GET http://127.0.0.1:8092/`
- Static navigation asset: `GET http://127.0.0.1:8092/js/components/nav.js`
- UI input: clicked the `Tools` navigation control in the local page
- Account test inputs:
  - anonymous: `null`
  - ordinary user: `role=USER`, no protected permissions
  - listener: `role=USER`, `MUSIC_READ`
  - writer: `role=USER`, `MUSIC_WRITE`
  - shared reader: `role=USER`, `SHARED_FOLDER_READ`
  - administrator: `role=ADMIN`, no separately supplied permissions

## Response Received

- `GET /` response status code: `200 OK`.
- `GET /js/components/nav.js` response status code: `200 OK`.
- UI result: the signed-out Tools dropdown opened and rendered the expected public destinations.
- The signed-out top-level navigation displayed `Feed`, `Explore`, `Messages`, and `Tools`; Music was not present there.
- Expanded signed-out Tools displayed, in order: `Raising Canes Box Index`, `VIN Decoder`, `What's For Lunch`, `ZIP Coordinates`.
- No signed-out Tools link was rendered for Back Office, Command Center, Music, or Shared Folder.
- The access projection tests returned effective Music read for administrator, `MUSIC_READ`, and `MUSIC_WRITE`; missing or failed account state returned no protected access.
- The administrator Tools projection returned, in order: `Back Office`, `Command Center`, `Music`, `Raising Canes Box Index`, `Shared Folder`, `VIN Decoder`, `What's For Lunch`, `ZIP Coordinates`.

## Pass / Fail

| Test case | Result | Reason |
| --- | --- | --- |
| Local page and navigation asset | PASS | Both requests returned HTTP 200. |
| Music removed from top level | PASS | The browser showed no top-level Music item. |
| Signed-out protected-link gating | PASS | No protected Tools destination was rendered. |
| Signed-out alphabetical order | PASS | The four public entries appeared in ascending label order. |
| Music reader/writer access | PASS | Both capabilities produced effective Music read access. |
| Administrator access | PASS | ADMIN received Music, Back Office, and Command Center without separate permission flags. |
| Fail-closed account state | PASS | Missing account data and profile-request failure project no protected access. |
| Focused regression tests | PASS | 23 tests passed with zero failures. |
| Complete JavaScript suite | PASS | 270 tests passed with zero failures. |
| Full Gradle check | PASS | Build completed successfully; 1,393 Java tests ran with zero failures and zero errors, with 3 skipped. |
| Runtime cleanup | PASS | PID 10476 stopped, port 8092 closed, and the isolated database was dropped. |

## Evidence

- TDD RED: the first focused run failed because `accountHasMusicRead` did not exist and the old menus still owned the destinations.
- TDD RED for stale authorization state: the navigation test failed before `accountNavigationAccess` existed.
- TDD GREEN: `node --test website/src/test/js/music.test.js website/src/test/js/nav-messages-link.test.js` completed with 23 passed and 0 failed.
- Complete JavaScript verification: `node --test website/src/test/js/*.test.js` completed with 270 passed and 0 failed in 543 ms.
- Full repository verification: `gradlew.bat :website:check --no-daemon` completed with `BUILD SUCCESSFUL` in 2 minutes 32 seconds.
- Java XML totals: 204 suites, 1,393 tests, 0 failures, 0 errors, 3 skipped.
- Browser evidence: the local Tools dropdown showed exactly the four public entries in alphabetical order and no protected entries.
- Cleanup evidence: the isolated database drop returned `{"dropped":1}` and the alternate listener was no longer present.

## Bugs / Follow-ups

No defect remains from the requested navigation change. Authenticated listener and administrator visibility is covered by deterministic account-projection and menu-builder tests because no local fixture credentials were used in the browser. Direct-route authorization was intentionally unchanged and remains the server-side enforcement boundary.
