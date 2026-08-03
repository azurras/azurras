# WFL Thumbs Voting Test Report

## Document Status

complete

## Story/Issue

Replace the What's for Lunch five-star rating system with binary thumbs-up/thumbs-down voting. Convert legacy ratings of 3-5 to `UP` and 1-2 to `DOWN`, use approval-weighted restaurant selection, update profile/list presentation to Void styling, and retain indexable restaurant profiles.

Related Builder work: `docs/work/2026-08-03-wfl-thumbs-voting.md`.

## Branch

- Repository: `azurras/christopherbell.dev`
- Branch: `codex/wfl-thumbs-voting`
- Commit under test: `fbab5e8816c66fd8c46147a95cf43f0832c3b341`
- Base: `origin/main` at `363bb986581c4d20df3434154844807ce88701e4`

## App / Environment

- App: `website` Spring Boot application
- Host: Windows development/production host
- Candidate profile: `local`
- Candidate port and base URL: `8094`, `http://localhost:8094`
- Candidate database: `christopherbell_dev_wfl_thumbs_voting_candidate`
- Invalid-migration database: `christopherbell_dev_wfl_thumbs_voting_invalid`
- Packaged artifact: `A:\Projects\christopherbell.dev-worktrees\wfl-thumbs-voting\website\build\libs\website.jar`
- Isolated worktree: `A:\Projects\christopherbell.dev-worktrees\wfl-thumbs-voting`
- Private Gradle home: `A:\Projects\christopherbell.dev-gradle-homes\wfl-thumbs-voting`
- Production listener on port 8080 was not changed during candidate testing.

## Local Run Details

The packaged candidate was started with the following effective command:

```powershell
java -jar A:\Projects\christopherbell.dev-worktrees\wfl-thumbs-voting\website\build\libs\website.jar `
  --spring.profiles.active=local `
  --server.port=8094 `
  --spring.mongodb.uri=mongodb://127.0.0.1:27017 `
  --spring.mongodb.database=christopherbell_dev_wfl_thumbs_voting_candidate `
  --command-center.enabled=false `
  --canes-box-tracker.enabled=false `
  --wfl.restaurant-import.monthly.enabled=false `
  --wfl.restaurant-of-the-day.enabled=false `
  --app.mail.enabled=false `
  --app.browser-security.public-base-url=http://localhost:8094 `
  --logging.file.name=.superpowers/sdd/2026-08-03-wfl-thumbs-voting/runtime-candidate-clean-20260803.log
```

- Candidate process: PID 69728
- Candidate log: `A:\Projects\christopherbell.dev-worktrees\wfl-thumbs-voting\.superpowers\sdd\2026-08-03-wfl-thumbs-voting\runtime-candidate-clean-20260803.log`
- Invalid-migration run used the same packaged artifact, port, and disabled background jobs against `christopherbell_dev_wfl_thumbs_voting_invalid`.
- Candidate PID 69728 was identity-checked, stopped after testing, and port 8094 was confirmed free.
- Both disposable databases were dropped after evidence capture. They contained only purpose-built test fixtures and are not recoverable; production data was not touched.

## Test Cases

### 1. Migration rejects invalid legacy data before mutation

Seeded ten legacy rating documents, including an invalid `rating: 6`, then started the candidate against the invalid-migration database.

Result: **PASS**. Startup failed at V013. All ten legacy documents remained legacy-shaped, no vote-shaped document was written, and the durable migration record was `FAILED`.

### 2. Migration converts valid legacy ratings and preserves constraints

Seeded nine valid legacy documents containing ratings 1 through 5 and started the candidate against the clean candidate database.

Result: **PASS**. Ratings 3, 4, and 5 became `UP`; ratings 1 and 2 became `DOWN`; no `rating` field remained. The migration produced six `UP` and three `DOWN` votes, retained the unique restaurant/account index, and recorded V013 as `APPLIED`.

### 3. Health, public API, redirect, sitemap, profile SEO, and zero-vote profile

Exercised readiness/liveness, top-liked/profile APIs, the retired top-rated route, sitemap, a voted profile, a zero-vote profile, and a missing profile.

Result: **PASS**. Health was `UP`; API aggregates were correct; the old route returned a permanent redirect; sitemap and canonical metadata used current URLs; structured data represented approval as 0-100; zero-vote structured aggregate data was omitted; missing profiles were non-indexable.

### 4. Authentication and strict vote mutation contract

Created and signed in a disposable local account through the real CSRF-protected signup/login flows. Exercised anonymous voting, valid votes, same-vote idempotence, vote changes, legacy numeric inputs, extra fields, and the removed rating endpoint.

Result: **PASS**. Anonymous mutation was forbidden. Valid `UP`/`DOWN` votes succeeded. Repeating the same vote did not add a document. Numeric/legacy/unknown inputs were rejected. The old rating endpoint was absent. The database retained exactly one vote per restaurant/account.

### 5. Approval-weighted restaurant selection

Issued 400 nearby-selection requests, each returning three restaurants, against fixtures spanning 100%, 75%, no-vote, and 0% approval states.

Result: **PASS**. Higher-approval restaurants appeared more frequently and low-approval restaurants less frequently, while no-vote restaurants remained eligible.

### 6. Authenticated desktop UI

At a 1440 by 900 viewport, signed in through the UI, opened the Alpha Tacos profile, changed the vote from `UP` to `DOWN`, favorited the restaurant, and opened Favorites and Top Liked.

Result: **PASS**. The profile/list pages used the Void palette, exposed accessible thumb buttons, reflected the selected vote with pressed state and summary updates, and showed the same controls on Favorites and Top Liked. The final Alpha summary was `60% liked · 3 up · 2 down`, with `Thumbs down` pressed and `Favorited` pressed.

### 7. Mobile UI and console

At a 390 by 844 viewport, opened the authenticated Alpha Tacos profile and measured the document, summary, and controls. Collected browser warnings/errors after the UI flows.

Result: **PASS**. Document scroll width was 375 at a 375 client width, summary white-space was `normal`, summary width fit its container, thumb buttons were 48 by 44 pixels, the page retained the Void background `rgb(7, 11, 16)`, and the browser warning/error log was empty.

### 8. Automated regression suites

Ran the focused Pester suite, Java tests, JavaScript tests, and Gradle `:website:check` from the isolated worktree.

Result: **PASS**. Pester passed 92/92, Java passed 1,656 tests with zero failures, JavaScript passed 336/336, and `:website:check` completed successfully.

## Data Sent

- Migration fixture: ten legacy `whatsforlunch_ratings` documents with one invalid `rating: 6` for preflight rejection.
- Valid migration fixture: nine legacy documents containing numeric ratings 1-5.
- Public requests:
  - `GET http://localhost:8094/actuator/health/readiness`
  - `GET http://localhost:8094/actuator/health/liveness`
  - `GET http://localhost:8094/api/v1/whats-for-lunch/restaurants/top-liked`
  - `GET http://localhost:8094/api/v1/whats-for-lunch/restaurants/rest-alpha`
  - `GET http://localhost:8094/wfl/top-rated`
  - `GET http://localhost:8094/sitemap.xml`
  - `GET http://localhost:8094/wfl/restaurants/rest-alpha`
  - `GET http://localhost:8094/wfl/restaurants/rest-echo`
  - `GET http://localhost:8094/wfl/restaurants/missing`
- Signup UI/form input: first name `Runtime`, last name `Voter`, username `runtime-voter`, email `runtime-voter@example.test`, and a disposable candidate-only password.
- Vote mutations, authenticated where noted:
  - anonymous `PUT` with `{"vote":"UP"}`
  - authenticated `PUT` with `{"vote":"UP"}` twice
  - authenticated `PUT` with `{"rating":5}`
  - authenticated `PUT` with `{"rating":null}`
  - authenticated `PUT` with a numeric vote
  - authenticated path-form vote change to `DOWN`
  - authenticated path-form request with an unknown extra field
  - request to the retired `/rating` mutation endpoint
- Selection input: 400 nearby-selection requests returning three restaurants each.
- UI input: profile sign-in; thumb-up click; thumb-down click; Favorite click; navigation to Favorites and Top Liked; mobile viewport 390 by 844.

## Response Received

- Representative running-app response: status code: 200 with response body `{"status":"UP"}` from readiness.
- Representative UI result: the Alpha profile changed to `60% liked · 3 up · 2 down` with `Thumbs down` pressed.
- Invalid preflight:
  - application startup failed at V013
  - legacy document count: 10
  - vote document count: 0
  - invalid document unchanged
  - V013 state: `FAILED`
  - checksum: `c10c2769b37044d866224770f7fb8b0877e02c2457c53d33ee25eeb879ab86f7`
- Valid migration:
  - V013 state: `APPLIED`
  - votes: 6 `UP`, 3 `DOWN`
  - legacy `rating` fields: 0
  - retained unique index: `restaurant_account_unique`
- Health:
  - readiness: HTTP 200, `{"status":"UP"}`
  - liveness: HTTP 200
- Public data:
  - Alpha profile API: 3 up, 1 down, count 4 before UI mutation
  - Echo profile API: 0 up, 0 down, count 0
  - Top Liked order: Bravo 100%/2 votes; Charlie 100%/1; Alpha 75%/4; Delta 0%/2
  - retired route: HTTP 308, `Location: /wfl/top-liked`
  - sitemap contained `/wfl/top-liked` and Alpha profile, and did not contain `/wfl/top-rated`
  - Alpha HTML: HTTP 200, canonical URL present, no `noindex`, `75% liked · 3 up · 1 down`
  - Alpha JSON-LD: `ratingValue: 75`, `bestRating: 100`, `worstRating: 0`, `ratingCount: 4`
  - Echo HTML: `No votes yet`; no JSON-LD `aggregateRating`
  - missing profile: HTTP 404 with `noindex`
- Vote contract:
  - anonymous vote: HTTP 403
  - first `UP`: HTTP 200, count 1, `myVote: UP`
  - repeated `UP`: HTTP 200, still count 1
  - legacy `rating: 5`: HTTP 400
  - legacy `rating: null`: HTTP 400
  - numeric vote: HTTP 400
  - retired rating endpoint: HTTP 404
  - change to `DOWN`: HTTP 200, count 1, `myVote: DOWN`
  - unknown extra input: HTTP 400
- Weighted sample appearances out of 400 three-choice draws:
  - Charlie, 100% with one vote: 231, 57.8%
  - Bravo, 100% with two votes: 227, 56.8%
  - Alpha, then 75%: 222, 55.5%
  - Foxtrot, no votes: 190, 47.5%
  - Echo, low approval: 168, 42.0%
  - Delta, 0%: 162, 40.5%
- Final UI/database state:
  - Alpha summary: `60% liked · 3 up · 2 down`
  - `Thumbs down` pressed; `Thumbs up` not pressed
  - `Favorited` pressed
  - exactly one Alpha/runtime-account vote document, value `DOWN`
  - exactly one Alpha/runtime-account favorite document
  - no browser warnings or errors

## Pass / Fail

**PASS**. All required migration, compatibility rejection, API, selection weighting, SEO, desktop UI, mobile UI, persistence, and automated regression checks passed against the packaged candidate. Production cutover remains a separate deployment phase.

## Evidence

- Candidate log: `A:\Projects\christopherbell.dev-worktrees\wfl-thumbs-voting\.superpowers\sdd\2026-08-03-wfl-thumbs-voting\runtime-candidate-clean-20260803.log`
- Task and review evidence: `A:\Projects\christopherbell.dev-worktrees\wfl-thumbs-voting\.superpowers\sdd\2026-08-03-wfl-thumbs-voting`
- Packaged JAR tested: `A:\Projects\christopherbell.dev-worktrees\wfl-thumbs-voting\website\build\libs\website.jar`
- Browser evidence came from accessible DOM snapshots, structured layout measurements, and browser warning/error logs in the Codex in-app browser.
- MongoDB evidence was captured with `mongosh` before the disposable databases were dropped.
- Candidate process identity and command line were inspected before stop; port 8094 was rechecked after stop.

## Bugs / Follow-ups

- No unresolved functional defects were found in candidate testing.
- The browser automation backend's `press` helper focused the native button but did not synthesize a click in this session; pointer activation and the native semantic button/ARIA contract were verified. Repository keyboard/accessibility tests remain green.
- Production migration and post-cutover verification are intentionally pending the merged, CI-approved artifact and the approved forward-only outage procedure.
