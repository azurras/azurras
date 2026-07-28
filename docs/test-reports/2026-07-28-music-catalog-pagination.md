# Music Catalog Pagination Test Report

## Document Status

complete

## Story/Issue

User-requested correction for the Music catalog showing only 98 tracks, plus numbered paging and confirmation that radio selects from the entire eligible catalog rather than the visible page.

## Branch

- Spoke branch: `codex/music-catalog-pagination`
- Spoke commits: `663b4a2f`, `1564b4d1`
- Pull request: `azurras/christopherbell.dev#1313`
- Production merge: `7d7d042c26d7bbabee2cdf0bc430127a0020e65e`

## App / Environment

- App: `christopherbell.dev` Spring Boot website
- Candidate profiles: `local,deploy-smoke`
- Candidate base URL: `http://127.0.0.1:8091`
- Candidate database: isolated local Mongo database `christopherbell_codex_music_pagination`
- Candidate Music scanner: disabled with `APP_MUSIC_ENABLED=false`
- Production base URL: `https://www.christopherbell.dev`
- Production listener: native Windows service on port 8080

## Local Run Details

The candidate was started from `A:\Projects\christopherbell.dev-worktrees\music-catalog-pagination` with:

```powershell
$env:GRADLE_USER_HOME='A:\Temp\gradle-music-pagination'
$env:SPRING_PROFILES_ACTIVE='local,deploy-smoke'
$env:SERVER_PORT='8091'
$env:SPRING_MONGODB_DATABASE='christopherbell_codex_music_pagination'
$env:APP_MUSIC_ENABLED='false'
.\gradlew.bat :website:bootRun --no-daemon
```

The process ran hidden with output under `A:\Temp\music-pagination-runtime`. The exact process tree started for the check was stopped afterward, and port 8091 was confirmed closed. Production port 8080 was not changed during candidate testing.

## Test Cases

1. Candidate root and Music page load on an alternate port.
2. Candidate Music page contains the new pagination mount point.
3. Candidate anonymous catalog access remains forbidden.
4. Full automated Java, JavaScript, sensor-runtime, and build verification passes.
5. Production deploys the exact merge SHA automatically and serves the new Music assets.
6. Production anonymous catalog and radio APIs remain forbidden.
7. The live radio candidate pool is independent of the 50-track visible page.

## Data Sent

- `GET http://127.0.0.1:8091/`
- `GET http://127.0.0.1:8091/music`
- `GET http://127.0.0.1:8091/api/music/2026-07-28/catalog?page=0&size=50` without authentication
- `GET https://www.christopherbell.dev/music` with a cache-busting query parameter
- `GET https://www.christopherbell.dev/api/music/2026-07-28/catalog?page=0&size=50` without authentication
- `GET https://www.christopherbell.dev/api/music/2026-07-28/radio` without authentication
- Read-only Mongo counts against `music_tracks` for present `READY` tracks and non-excluded radio candidates

## Response Received

- Candidate root: HTTP 200.
- Candidate Music page: HTTP 200 and `id="music-pagination"` present.
- Candidate anonymous catalog: HTTP 403.
- Production Music page: HTTP 200 with asset version `7d7d042c26d7bbabee2cdf0bc430127a0020e65e`.
- Production Music JavaScript: HTTP 200 and includes the new paged-catalog request path.
- Production anonymous catalog: HTTP 403.
- Production anonymous radio: HTTP 403.
- Production services `ChristopherBellDev`, `ChristopherBellMediaWorker`, `cloudflared`, and `MongoDB`: Running and Automatic.
- Production port 8080: listening after a clean listener PID rotation.
- Live catalog at verification time: 1,068 present READY tracks, 32 probe failures, and 1,068 non-excluded radio candidates. The scheduled scanner had just written another batch and continues adding remaining supported on-disk files.

## Pass / Fail

- Candidate startup and page wiring: PASS.
- Pagination markup and production asset delivery: PASS.
- Anonymous authorization boundary: PASS.
- Radio independence from visible page: PASS; the service requests up to 10,000 eligible tracks and the live candidate count was 1,068 while the page size is 50.
- Automated verification: PASS; `:website:check` completed successfully, the focused Java suite passed 14 tests, and the JavaScript suite passed 252 tests.
- Pull-request and post-merge CI: PASS across Windows, Linux, macOS, dependency review, and CodeQL.

## Evidence

- Full local command: `.\gradlew.bat :website:check --no-daemon` -> `BUILD SUCCESSFUL in 1m 29s`.
- Focused command: `.\gradlew.bat :website:test --tests MusicCatalogTest --tests MusicLibraryServiceTest --tests MusicReadControllerTest --tests MusicRadioServiceTest :website:jsTest --no-daemon` -> 14 Java tests and 252 JavaScript tests passed.
- Pull request: https://github.com/azurras/christopherbell.dev/pull/1313
- Main CI run: https://github.com/azurras/christopherbell.dev/actions/runs/30390384670
- Main CodeQL run: https://github.com/azurras/christopherbell.dev/actions/runs/30390384697
- Production release observed at 2026-07-28 14:09 America/Chicago.

## Bugs / Follow-ups

- No release-blocking defect remains.
- An authenticated visual walkthrough was not performed because no test login session was introduced into the automated runtime check. Controller, service, and browser-module tests cover authenticated response paging, and the user can now verify the live account view.
- The scanner intentionally probes at most 100 changed files per five-minute pass. Therefore, supported files not yet indexed join the catalog and the full radio pool incrementally rather than all at once.
