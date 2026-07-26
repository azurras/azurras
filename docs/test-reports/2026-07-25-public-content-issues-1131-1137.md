# Public Content Issues 1131-1137 Test Report

## Document Status

complete

## Story/Issue

GitHub issues `cbell504/website#1131`, `#1132`, `#1133`, `#1134`, `#1135`, `#1136`, and `#1137`.

## Branch

Spoke branch `codex/public-content-1131-1137` from `origin/main` commit `b6c361d1d916337679a37f04caa46c3475215e71`. The commit, pull request, CI, merge, and production sections will be appended after publication.

## App / Environment

- App: `christopherbell.dev` Spring Boot application.
- Worktree: `A:\Projects\christopherbell.dev-worktrees\public-content-1131-1137`.
- Profile and alternate base URL: `local`, `http://localhost:8090`.
- Environment: `SPRING_PROFILES_ACTIVE=local`, `SERVER_PORT=8090`, `APP_PUBLIC_BASE_URL=http://localhost:8090`, isolated `GRADLE_USER_HOME=A:\Temp\gradle-public-content-1131-1137`.
- Production safety: production PID `26680` remained on port `8080`. The branch ran only on `8090`, its final process PID `22656` was stopped, the port was proven free, and production `/` returned `200` with body length `4035` afterward.

## Local Run Details

The alternate-port application was started from the isolated worktree with:

```powershell
$env:SPRING_PROFILES_ACTIVE='local'
$env:SERVER_PORT='8090'
$env:APP_PUBLIC_BASE_URL='http://localhost:8090'
$env:GRADLE_USER_HOME='A:\Temp\gradle-public-content-1131-1137'
.\gradlew.bat :website:bootRun --no-daemon
```

Acceptance requests were captured on `2026-07-25` between approximately `21:20` and `21:31 -05:00`. The final static and automated verification completed at `21:31 -05:00`.

## Test Cases

1. Prove the old blog/photo routes, unsupported blog tags, invalid archive links, insecure archive images, and Bootstrap CDN references fail focused RED contracts.
2. Prove anonymous blog list/detail and photo list APIs return the standard response envelope.
3. Prove equivalent POST requests remain denied.
4. Load the public blog component and verify configured content renders as text.
5. Load the photo gallery, verify the usage link, and navigate to `/photos/usage`.
6. Verify gallery alt fallback behavior with description, name, and generic content text.
7. Load The Bell and Tony pages; inspect links, image sources, image count, favicon, and console output.
8. Fetch the self-hosted Bootstrap 5.3.3 CSS and JavaScript assets anonymously and verify the main stylesheet imports the pinned CSS.
9. Scan all resource files recursively for Bootstrap CDN URLs, The Bell HTTP image sources, invalid archive links, and missing local archive assets.
10. Run focused and complete Java/JavaScript suites, JavaScript syntax checks, dependency insight, `git diff --check`, `bootJar`, and repository `check` lifecycle.

## Data Sent

- Anonymous `GET` requests to `/blog`, `/api/blog/v1/posts`, `/photos`, `/api/photo/v1`, `/photos/usage`, `/thebell`, `/thebell/tony`, `/webjars/bootstrap/5.3.3/css/bootstrap.min.css`, and `/webjars/bootstrap/5.3.3/js/bootstrap.bundle.min.js`.
- Anonymous `POST` requests to `/api/blog/v1/posts`, `/api/photo/v1`, and both exact Bootstrap WebJar asset paths.
- Browser navigation to `http://127.0.0.1:8090/blog`, `/photos`, `/photos/usage`, and `/thebell/tony` in an isolated in-app browser tab.
- No account credentials, authenticated mutations, or issue attachments were used.

## Response Received

- `GET /blog`: `200 text/html`, length `2912`.
- `GET /api/blog/v1/posts`: `200 application/json`, length `280`, `success=true`, with `payload.posts` containing the configured `test blog` post.
- `GET /photos`: `200 text/html`, length `2776`.
- `GET /api/photo/v1`: `200 application/json`, length `73`, body `{"messages":null,"payload":{"images":[]},"requestId":null,"success":true}`.
- `GET /photos/usage`: `200 text/html`, length `2475`.
- `GET /thebell`: `200 text/html`, length `7058`; `GET /thebell/tony`: `200 text/html`, length `3026`.
- WebJar CSS: `200 text/css`, length `232800`, identifies Bootstrap `v5.3.3`.
- WebJar JavaScript: `200 text/javascript`, length `80721`, identifies Bootstrap `v5.3.3`.
- All four equivalent POST probes returned `403` with empty response bodies.
- Local CSP narrowed to `script-src 'self'` and removed jsDelivr from `style-src`; `X-Frame-Options` remained `SAMEORIGIN`.
- Browser blog text was `test blogAuthor: TestTest Content`; the final screenshot showed the post rendered inside the public blog page with Bootstrap styling and zero console messages.
- The gallery exposed `href="/photos/usage"`; the usage page rendered its consent text and zero console messages.
- Tony exposed three article images, no empty/numeric links, no insecure image sources, the versioned local `K-On.jpg` favicon, and zero console messages.
- The active `main.css` stylesheet contained an imported `/webjars/bootstrap/5.3.3/css/bootstrap.min.css` rule.

## Pass / Fail

- PASS: public blog/photo APIs use the current versioned routes and standard response envelopes.
- PASS: API/WebJar public access is GET-only; mutation-shaped probes are denied.
- PASS: the blog component renders after both raw-envelope and shared-`fetchJson` payload normalization.
- PASS: unsupported tag controls and polling are removed.
- PASS: gallery envelope parsing and alt fallback contracts are executable; the configured local gallery is empty, so live alt attributes were validated by unit partition rather than fixture mutation.
- PASS: the photography usage route is public and linked.
- PASS: archive invalid links, missing favicon, insecure image sources, and stray markers are removed.
- PASS: Bootstrap is pinned and self-hosted with no jsDelivr Bootstrap reference in application resources.
- PASS: alternate-port cleanup and production continuity checks succeeded.

## Evidence

- Initial focused Node RED: 4 tests failed for missing current photo API/helper behavior, archive hygiene, and local Bootstrap delivery.
- Definitive focused Java RED: 5 behavioral failures for anonymous blog/photo access, public matchers, and `/photos/usage`.
- Browser-discovered RED after the first automated GREEN: the blog host contained only `<div class="blogPosts"></div>` because `fetchJson` had already unwrapped `payload`; the added unwrapped-payload assertions failed before the one-line-per-normalizer fix and passed afterward.
- Focused Java command covering `BlogControllerTest`, `PhotoControllerTest`, `SecurityConfigTest`, and `ViewControllerTest`: 29 passed.
- Focused Node command `node --test website/src/test/js/public-content.test.js`: 4 passed after witnessed RED failures.
- Final full Java command `:website:test`: 108 suites, 1002 tests, 0 failures, 3 skipped.
- Final full JavaScript command `:website:jsTest`: 199 passed, 0 failed.
- Final `:website:check`: `BUILD SUCCESSFUL`; included `bootJar`, full tests, JavaScript tests, and `verifySensorRuntime`.
- `dependencyInsight` selected exactly `org.webjars:bootstrap:5.3.3` on `runtimeClasspath`; the repository has no Gradle verification metadata and relies on its existing Dependency Review CI workflow plus Dependabot.
- `node --check` passed for all changed JavaScript and the new Node test; `git diff --check` passed.
- Recursive resource scans found zero Bootstrap jsDelivr references and zero The Bell `src="http://` values.

## Bugs / Follow-ups

- Fixed during local browser testing: the first implementation double-unwrapped the API response because `fetchJson` already returns `data.payload`. The regression now covers both the raw response envelope and the shared-helper payload shape.
- The local profile's unrelated startup-time OpenStreetMap catch-up listener fetched candidates and logged a duplicate-key write failure for an existing normalized restaurant name. This did not affect the tested public-content routes or production availability, but future alternate-port runs should suppress scheduled/import startup work or use an isolated database when the repository provides that boundary.
- The worktree continues to show the pre-existing `gradlew.bat` LF-to-CRLF checkout-only difference. It is excluded from the spoke commit.
