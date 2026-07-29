# christopherbell.dev Void Public Discovery Test Report

## Document Status

complete

## Story/Issue

Approved Void public-discovery release: normalized topics, New/Fading/Revived/Topics/People discovery, public Explore/topic pages, new-account mutation limits, and seven-day login tokens. Pull request: https://github.com/azurras/christopherbell.dev/pull/1315

## Branch

- Branch: `codex/void-public-discovery`
- Commit under final verification: `da602feb5944c074135e6ba4b974ac34c5d284ba`
- Base: `main` at `7e958e737b34563d6d49a078243437d5fa9e3377`

## App / Environment

- App: `christopherbell.dev` Spring Boot website
- Profile: `local`
- Alternate port and base URL: `8081`, `http://127.0.0.1:8081`
- Mongo URI: `mongodb://127.0.0.1:27017`
- Isolated database: `christopherbell_void_discovery_20260728_01`
- Explicit environment: `SPRING_PROFILES_ACTIVE=local`, `SERVER_PORT=8081`, `SPRING_MONGODB_DATABASE=christopherbell_void_discovery_20260728_01`, `APP_MAIL_ENABLED=false`
- Production listener: port `8080`, PID `25764`; never stopped or modified
- Production fixture query before and after testing: zero documents matching `_id: /^e2e-void-/`

## Local Run Details

The app was started from `A:\Projects\christopherbell.dev-worktrees\void-public-discovery` with the explicit environment above and:

```powershell
.\gradlew.bat :website:bootRun --no-daemon --console=plain
```

The successful alternate-port Java process was PID `46824`. Standard output and error were captured in `A:\Temp\void-discovery-bootrun-2.out.log` and `A:\Temp\void-discovery-bootrun-2.err.log`. After runtime and browser checks, PID `46824` was stopped and port `8081` was confirmed closed. The exact isolated database was dropped successfully. Port `8080` remained listening on PID `25764`.

## Test Cases

1. Start the complete Spring application with both Mongo URI and database set explicitly.
2. Load the public Explore shell anonymously.
3. Read New, Fading, Revived, Topics, People, and one topic endpoint anonymously.
4. Confirm bounded first pages, continuation cursor behavior, and `no-store` caching.
5. Load all five Explore panels in a real browser.
6. Confirm one panel family can fail without blanking an unaffected panel.
7. Load 12 more New items and confirm the section grows from 12 to 24 without replacing other sections.
8. Navigate through `#Music` to `/void/topic/music` and confirm six matching active posts render.
9. Run the full repository verification after the runtime-discovered startup fix.
10. Confirm test fixtures never entered the production database and clean up the isolated database.

## Data Sent

- Seeded six synthetic ACTIVE USER accounts and 30 synthetic active root posts in the isolated database only.
- Posts covered six normalized topics, 15 revival timestamps, staggered creation times, and staggered expiration times.
- Anonymous GET requests:
  - `/void/explore`
  - `/api/posts/2026-07-28/discovery/new?size=12`
  - `/api/posts/2026-07-28/discovery/fading?size=12`
  - `/api/posts/2026-07-28/discovery/revived?size=12`
  - `/api/posts/2026-07-28/discovery/topics?size=12`
  - `/api/posts/2026-07-28/discovery/people`
  - `/api/posts/2026-07-28/discovery/topic/music?size=12`
- Browser actions: open Explore, wait for populated content, click New `Load more`, click the `#Music` topic link.
- No credentials, real accounts, production writes, or bearer tokens were used.

## Response Received

- Explore status code: 200; title `CB | Explore the Void`; 6,283-byte server-rendered shell.
- New status code: 200; 12 items; continuation cursor present; `Cache-Control: no-store`.
- Fading status code: 200; 12 items; continuation cursor present; `Cache-Control: no-store`.
- Revived status code: 200; 12 items; continuation cursor present; `Cache-Control: no-store`.
- Topics status code: 200; six items; `Cache-Control: no-store`.
- People status code: 200; six active public-safe account suggestions; `Cache-Control: no-store`.
- Topic Music status code: 200; six active matching roots; `Cache-Control: no-store`.
- Browser UI state rendered all five sections, account links, lifespan labels, topic chips, people cards, and section-local Load more controls.
- Browser pagination increased the New section from 12 to 24 cards.
- Browser topic navigation ended at `http://127.0.0.1:8081/void/topic/music`, title `CB | #music`, with six post cards.
- Failure-isolation UI state left Topics populated while failed post/people requests displayed section-local `This section could not load. Try again.` states.
- Final `:website:check`: BUILD SUCCESSFUL in 1m 36s; 1,304 Java tests with zero failures/errors and 265 JavaScript tests with zero failures/errors; runtime-sensor verification passed.

## Pass / Fail

- PASS â€” complete application startup on isolated port/database.
- PASS â€” all public discovery endpoints and cache policy.
- PASS â€” active-only bounded paging and cursor behavior.
- PASS â€” populated desktop Explore and topic experiences.
- PASS â€” independent section failure handling and retry presentation.
- PASS â€” browser pagination and topic navigation.
- PASS â€” full automated repository check after the runtime fix.
- PASS â€” production safety and test-data cleanup.

## Evidence

- Full verification command: `$env:GRADLE_USER_HOME='A:\Temp\gradle-void-discovery'; .\gradlew.bat :website:check --no-daemon --console=plain`
- Java XML evidence: `website/build/test-results/test/TEST-*.xml` (1,304 tests, 0 failures, 0 errors, 3 skipped).
- JavaScript XML evidence: `website/build/test-results/jsTest/results.xml` (265 tests, 0 failures, 0 errors).
- Runtime logs: `A:\Temp\void-discovery-bootrun-2.out.log`, `A:\Temp\void-discovery-bootrun-2.err.log`.
- Browser DOM snapshots captured the full Explore and topic structures; a desktop screenshot confirmed the responsive Void visual treatment.
- Final cleanup output: database drop `{ ok: 1, dropped: 'christopherbell_void_discovery_20260728_01' }`, production fixture count `0`, listeners `port=8080 pid=25764` only.

## Bugs / Follow-ups

Runtime startup initially failed because the two new hand-written `@Repository` classes were `final`, preventing Spring exception-translation proxies. A regression test failed against the original declarations, both repositories were made proxyable, the test passed, the complete app then started, and the final full check passed. No unresolved release-blocking defect remains from local verification.

The initial hand-written test fixture also used hyphenated topic names, which are invalid by the documented hashtag grammar. The isolated fixture was corrected to underscore forms; this was test-data cleanup, not an application defect.
