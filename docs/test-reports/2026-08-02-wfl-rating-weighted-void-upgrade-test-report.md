# WFL Rating-Weighted Void Upgrade Test Report

## Document Status

complete

## Story/Issue

[WFL rating-weighted Void upgrade work record](../work/2026-08-02-christopherbell-dev-wfl-rating-weighted-void-upgrade.md)

## Branch

- Spoke repository: `A:\Projects\christopherbell.dev-worktrees\wfl-rating-weighted-void`
- Branch: `codex/wfl-rating-weighted-void`
- Commit under test: `17583448ed2a4d6f425f766157a357a613037d17`
- Base: `origin/main` at `1d1b322dc1667e48bc0230009a3fe79fce0a1b90`

## App / Environment

- App: `christopherbell.dev` Spring Boot website
- Profile: `local`
- Candidate base URL: `http://127.0.0.1:8081`
- Candidate process: Java PID `60080`
- Production isolation: existing `:8080` listener remained on PID `57904`
- Database: local MongoDB at `mongodb://localhost:27017`; driver connected successfully
- Mail: disabled with `APP_MAIL_ENABLED=false`
- Gradle home: `C:\Users\Christopher\AppData\Local\Codex\gradle\wfl-rating-weighted-void`

## Local Run Details

The candidate was started from the isolated worktree with a hidden PowerShell `Start-Process` invocation of:

```powershell
.\gradlew.bat :website:bootRun --no-daemon
```

The process environment set `GRADLE_USER_HOME` to the private task directory, `SPRING_PROFILES_ACTIVE=local`, `SERVER_PORT=8081`, `SPRING_MONGODB_URI=mongodb://localhost:27017`, `APP_MAIL_ENABLED=false`, a task-only local JWT secret, and `GIT_COMMIT=17583448ed2a4d6f425f766157a357a613037d17`.

Logs were captured at:

- `C:\Users\Christopher\AppData\Local\Temp\codex-wfl-rating-weighted-void\bootrun.out.log`
- `C:\Users\Christopher\AppData\Local\Temp\codex-wfl-rating-weighted-void\bootrun.err.log`

Spring Boot reported `Started Application in 4.935 seconds`. After verification, PID `60080` was stopped, `:8081` no longer listened, and production `:8080` still listened on PID `57904`.

## Test Cases

1. Verify candidate liveness, readiness, and WFL page delivery on the alternate port.
2. Exercise today's persisted public picks.
3. Exercise fresh rating-weighted nearby selection using browser coordinates.
4. Exercise fresh rating-weighted nearby selection using a ZIP code.
5. Use the page UI to submit a ZIP and render three restaurant cards.
6. Verify the desktop Void decision console, equal card geometry, disclosure, and absence of ranking badges.
7. Verify the 390 by 844 mobile breakpoint, single-column cards, and absence of horizontal overflow.
8. Verify the dedicated WFL stylesheet does not load on the neighboring Top Rated page.
9. Verify the browser console has no warnings or errors.
10. Run focused and repository-wide automated verification supporting the runtime checks.

## Data Sent

### HTTP requests

- `GET http://127.0.0.1:8081/actuator/health/liveness`
- `GET http://127.0.0.1:8081/actuator/health/readiness`
- `GET http://127.0.0.1:8081/wfl`
- `GET http://127.0.0.1:8081/api/whatsforlunch/restaurant/2026-05-17/today`
- `GET http://127.0.0.1:8081/api/whatsforlunch/restaurant/2026-05-17/nearby?latitude=33.0782&longitude=-96.8089&radiusMiles=20&useSavedPreferences=false`
- `GET http://127.0.0.1:8081/api/whatsforlunch/restaurant/2026-05-17/nearby/zip/75024?radiusMiles=20&useSavedPreferences=false`

### Browser UI input

- Opened `http://127.0.0.1:8081/wfl` in Chrome.
- Entered ZIP code `75024` in the labelled ZIP input.
- Activated the unique `Use ZIP` button.
- Inspected the default 1276 by 1270 viewport and a temporary 390 by 844 viewport.
- Opened `http://127.0.0.1:8081/wfl/top-rated` to confirm stylesheet isolation.

## Response Received

### Health and page responses

- HTTP response status code: 200 for every health and page request below.
- Liveness: HTTP `200`, `application/vnd.spring-boot.actuator.v3+json`, body length 15.
- Readiness: HTTP `200`, `application/vnd.spring-boot.actuator.v3+json`, body length 15.
- `/wfl`: HTTP `200`, `text/html; charset=UTF-8`, body length 3161.

### Restaurant API responses

- Today's picks: HTTP `200`, `success=true`, three restaurant records.
- Coordinate nearby picks: HTTP `200`, `success=true`, three restaurant records: First Chinese BBQ-Richardson, 1418 Coffeehouse, and Sushi Poki.
- ZIP nearby picks: HTTP `200`, `success=true`, three restaurant records: Dunkin' Donuts, Black Agave, and Nuno's Tacos & Vegmex Grill.

### Browser response

- The ZIP submission rendered exactly three unranked cards with real addresses, including `3421 East Renner Road, Plano, TX, 75074`, `7949 Walnut Hill Lane, Dallas, TX, 75230`, and `6959 Lebanon Road, Frisco, TX, 75034`.
- The disclosure rendered visibly: `Ratings influence the draw. Every eligible restaurant stays in the mix.`
- Desktop card rectangles were all 373 pixels wide and 336 pixels high at distinct equal-grid positions.
- Desktop and mobile horizontal overflow were both zero pixels.
- The mobile viewport placed all three cards at the same 309-pixel width in one column.
- DOM checks found one H1 and zero `.lunch-pick-rank` elements.
- The page background computed to `rgb(7, 11, 16)`.
- The Top Rated page retained `bodyClass=site-page lunch-page`, loaded zero `whats-for-lunch.css` links, and had zero horizontal overflow.
- Browser console query returned an empty list for warning and error entries.

## Pass / Fail

| Test case | Result | Reason |
| --- | --- | --- |
| Candidate health and WFL delivery | Pass | All three requests returned HTTP 200. |
| Today's persisted picks | Pass | API returned `success=true` and three records. |
| Coordinate nearby selection | Pass | API returned `success=true` and three fresh records. |
| ZIP nearby selection | Pass | API returned `success=true` and three fresh records. |
| ZIP UI flow | Pass | Three addressed restaurant cards rendered after the submitted ZIP. |
| Desktop Void console | Pass | Three equal cards, disclosure, Void palette, and no ranking badges were observed. |
| Mobile responsive layout | Pass | One-column 309-pixel cards rendered with zero horizontal overflow. |
| Stylesheet isolation | Pass | Top Rated did not load the dedicated Picks stylesheet. |
| Browser diagnostics | Pass | No browser console warnings or errors were captured. |
| Automated support | Pass | Focused tests and full `:website:check` completed successfully. |

## Evidence

- Focused command: `.\gradlew.bat :website:test --tests "*RatingWeightedRestaurantSelectorTest" --tests "*RestaurantRatingQueryRepositoryTest" --tests "*RestaurantServiceTest" :website:jsTest --no-daemon`
- Focused result: 8 selector tests, 3 rating-query tests, 60 service tests, and 313 JavaScript tests passed.
- JavaScript syntax: `node --check website/src/main/resources/static/js/whats-for-lunch.js` passed.
- Full command: `.\gradlew.bat :website:check --no-daemon`
- Full result: `BUILD SUCCESSFUL` in 2 minutes 59 seconds, including Java, 313 JavaScript tests, 150 Pester tests, boot JAR, deployment build context, sensors, and static fingerprinting.
- Runtime process and listener checks recorded candidate PID `60080` on `:8081` and production PID `57904` on `:8080`.
- Full-page desktop and mobile screenshots were inspected in the in-app browser during this run.
- Application logs record Tomcat starting on port 8081 and MongoDB connecting to localhost.

## Bugs / Follow-ups

No blocking defect was found. The candidate was stopped cleanly after the pass. Production deployment and post-deployment verification remain delivery steps, not test gaps in this local report.
