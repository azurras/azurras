# Restaurant Profile Void SEO Test Report

## Document Status

complete

## Story/Issue

Direct user request: apply the Void presentation to restaurant profile pages and make valid restaurant profiles indexable by search engines without exposing member-only or audit data.

## Branch

- Repository: `azurras/christopherbell.dev`
- Branch: `codex/restaurant-profile-void-seo`
- Commit under test: `4535eb0ecad0711e3a509f4f37ec31230cc50d6a`

## App / Environment

- App: christopherbell.dev Spring Boot website
- Candidate base URL: `http://127.0.0.1:8094`
- Candidate process: Java PID `17680`
- Candidate database: `christopherbell_dev_restaurant_profiles_void_seo` on MongoDB at `127.0.0.1:27017`
- Production safety boundary: existing port `8080` listener remained PID `55848`
- Fixtures: one complete public restaurant, one sparse restaurant with an unsafe website value, and one absent restaurant ID
- Browser viewports: desktop `1440x900`; mobile `390x844`

## Local Run Details

The packaged candidate was run from the isolated worktree with:

```powershell
java -jar website/build/libs/website.jar --server.port=8094 --spring.mongodb.uri=mongodb://127.0.0.1:27017 --spring.mongodb.database=christopherbell_dev_restaurant_profiles_void_seo
```

The candidate emitted normal Spring Boot output to the launching terminal. A second diagnostic instance on port `8095` was used briefly with detailed actuator health enabled to distinguish transient readiness initialization from a database failure. The final candidate process on port `8094` was stopped after testing. Port `8094` was verified free, the exact isolated database was dropped, and production port `8080` was rechecked at PID `55848`.

## Test Cases

### 1. Complete public restaurant profile

- Sent: `GET /restaurants/codex-profile-complete-20260802`
- Received: HTTP `200`; server-rendered heading, address, cuisine, rating summary, phone, website, directions, canonical URL, and schema.org `Restaurant` JSON-LD.
- Result: PASS. The response was indexable and contained no `noindex` directive.

### 2. Sparse profile and unsafe optional data

- Sent: `GET /restaurants/codex-profile-sparse-20260802`
- Received: HTTP `200`; server-rendered name and `No ratings yet`; unsafe `javascript:` website was omitted. JSON-LD omitted unavailable address, rating, and website properties.
- Result: PASS. Missing or unsafe optional fields did not create broken or dangerous output.

### 3. Missing profile

- Sent: `GET /restaurants/codex-profile-missing-20260802`
- Received: HTTP `404`; `robots` metadata was `noindex,nofollow`; no restaurant JSON-LD or profile details were emitted.
- Result: PASS. Invalid profiles are not indexable.

### 4. Search-engine discovery

- Sent: `GET /robots.txt` and `GET /sitemap.xml`
- Received: HTTP `200` for both; robots allowed public crawling and referenced the canonical sitemap; sitemap contained both persisted fixture profile URLs.
- Result: PASS. Valid profile pages are discoverable by crawlers.

### 5. Public/private response boundary

- Sent: raw HTTP requests for the complete and sparse profiles, whose database fixtures included private creator and modifier sentinel values.
- Received: neither private sentinel, member rating, favorite state, nor audit fields appeared in HTML or JSON-LD.
- Result: PASS. Only the immutable public profile page model reached the view.

### 6. Anonymous browser experience

- Sent: desktop and mobile browser navigation to complete and sparse profile URLs.
- Received: Void-styled responsive profile, semantic public content, sign-in fallback for personal controls, no horizontal overflow, and no browser console errors.
- Result: PASS. Anonymous rendering required no personal-data fetch and remained usable at both viewports.

### 7. Authenticated personal controls

- UI input: created isolated account `codex_profile_test_20260802`, signed in, selected rating `5`, and pressed `Favorite`.
- Received: public aggregate changed to `4.7/5 from 3 ratings`; personal rating displayed `5/5`; favorite control changed to `Favorited` with `aria-pressed=true`.
- Result: PASS. Progressive enhancement preserved the public page while adding member-only state.

### 8. Stale authenticated session

- UI input: deleted the exact isolated browser session and reloaded the complete profile.
- Received: public profile and rating remained visible; sign-in fallback returned; member controls disappeared; no red error or console error appeared.
- Result: PASS. A personal-data `401` did not erase or replace public content.

### 9. Keyboard and responsive accessibility

- UI input: keyboard Tab navigation; desktop `1440x900`; mobile `390x844`.
- Received: a visible `3px` solid gold focus outline with `3px` offset; two-column desktop profile; one-column mobile profile; rating signal fit without overflow; mobile actions used full width.
- Result: PASS.

## Data Sent

- HTTP methods: public `GET` requests to liveness, readiness, robots, sitemap, complete profile, sparse profile, and missing profile endpoints.
- Browser inputs: account registration and sign-in form values in the isolated database; rating button `5`; favorite toggle; keyboard Tab.
- Fixture IDs: `codex-profile-complete-20260802`, `codex-profile-sparse-20260802`, and `codex-profile-missing-20260802`.
- Host and port: `127.0.0.1:8094` only for candidate runtime interactions.

## Response Received

- `/actuator/health/liveness`: status code: 200.
- `/actuator/health/readiness`: initially transient status code: 503 during startup, then repeated status code: 200; diagnostic health details confirmed MongoDB and readiness state `UP`.
- `/robots.txt`: status code: 200, `Allow: /`, canonical sitemap URL.
- `/sitemap.xml`: status code: 200, including both persisted restaurant profile URLs.
- Complete profile: status code: 200, one canonical link, conditional complete JSON-LD, no private sentinels.
- Sparse profile: status code: 200, safe sparse HTML and JSON-LD, no unsafe website link.
- Missing profile: status code: 404, `noindex,nofollow`, no JSON-LD.
- Browser console errors after anonymous, authenticated, mutation, stale-session, desktop, and mobile checks: `[]`.

## Pass / Fail

PASS. Every planned HTTP and browser case passed. The implementation provides a complete server-rendered public profile, crawler discovery, conditional structured data, scoped Void styling, responsive and keyboard-accessible behavior, and isolated member controls. The only observed build interruption was a Windows file lock from the still-running candidate JAR; after stopping only PID `17680`, the exact-commit full check passed.

## Evidence

- Desktop screenshot: `C:\Users\Christopher\.codex\visualizations\2026\08\02\019fc417-f979-7cd2-bd22-8dc1ada009e6\restaurant-profile-desktop.png`
- Mobile screenshot: `C:\Users\Christopher\.codex\visualizations\2026\08\02\019fc417-f979-7cd2-bd22-8dc1ada009e6\restaurant-profile-mobile.png`
- Raw HTML assertions parsed the JSON-LD and verified canonical, robots, public content, optional-field omission, and private-sentinel absence.
- Browser computed-layout evidence verified desktop columns `429.188px / 643.812px`, mobile one-column flow, zero page overflow, and a non-wrapping rating signal.
- Automated supporting evidence: `./gradlew.bat :website:check --rerun-tasks --no-daemon --console=plain` completed `BUILD SUCCESSFUL in 3m 14s` with 21 tasks executed.
- JavaScript suite: 320 tests passed during verification.
- Windows production-script suite: 74 tests passed, 0 failed.
- Cleanup evidence: `DROPPED_DB=christopherbell_dev_restaurant_profiles_void_seo`, `PORT_8094_FREE=True`, `PRODUCTION_8080_PID=55848`.

## Bugs / Follow-ups

No open defects or acceptance gaps. Visual review found and corrected an initial desktop rating wrap and a mobile overflow before final acceptance. The isolated test account, session, rating, favorites, and fixtures were removed with the isolated database.
