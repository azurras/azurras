# Public Content Issues 1131-1137 Test Report

## Document Status

complete

## Story/Issue

GitHub issues `cbell504/website#1131`, `#1132`, `#1133`, `#1134`, `#1135`, `#1136`, and `#1137`.

## Branch

Spoke branch `codex/public-content-1131-1137` from `origin/main` commit `b6c361d1d916337679a37f04caa46c3475215e71`. Implementation commits were `ea54749730df97f2bfc920271c8463eb826e3f2f`, `4108c5c6f5adf5877f247c2cff4cf543fd7eb1cd`, and `f5120784bf4763cbd57666839307be24d209198a`. Pull request [azurras/christopherbell.dev#1251](https://github.com/azurras/christopherbell.dev/pull/1251) was squash-merged to `main` as `4b82116a0ed489c74eed144a478f1b3a3944ada2`.

## App / Environment

- App: `christopherbell.dev` Spring Boot application.
- Worktree: `A:\Projects\christopherbell.dev-worktrees\public-content-1131-1137`.
- Profile and alternate base URL: `local`, `http://localhost:8090`.
- Environment: `SPRING_PROFILES_ACTIVE=local`, `SERVER_PORT=8090`, `APP_PUBLIC_BASE_URL=http://localhost:8090`, isolated `GRADLE_USER_HOME=A:\Temp\gradle-public-content-1131-1137`.
- Production safety: production PID `26680` remained on port `8080` during alternate-port testing. The branch ran only on `8090`, its final process PID `22656` was stopped, the port was proven free, and production `/` returned `200` with body length `4035` afterward. After merge, the automatic deployment performed its guarded candidate validation and replaced the live Java listener with PID `29012`.

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
- Initial `GET /api/photo/v1`: `200 application/json` with an empty `payload.images`, which exposed the configuration-binding defect during review. After correction, the same endpoint returned `success=true` with all 12 configured photos; the first was `The River Walk - San Antonio` at `/images/photos/IMG_0072.jpeg`.
- `GET /photos/usage`: `200 text/html`, length `2475`.
- `GET /thebell`: `200 text/html`, length `7058`; `GET /thebell/tony`: `200 text/html`, length `3026`.
- WebJar CSS: `200 text/css`, length `232800`, identifies Bootstrap `v5.3.3`.
- WebJar JavaScript: `200 text/javascript`, length `80721`, identifies Bootstrap `v5.3.3`.
- All four equivalent POST probes returned `403` with empty response bodies.
- Local CSP narrowed to `script-src 'self'` and removed jsDelivr from `style-src`; `X-Frame-Options` remained `SAMEORIGIN`.
- Browser blog text was `test blogAuthor: TestTest Content`; the final screenshot showed the post rendered inside the public blog page with Bootstrap styling and zero console messages.
- The final gallery rendered all 12 configured images and exposed `href="/photos/usage"`; the first three alt values used their photo names instead of the `n/a` sentinel, the one real description remained preferred, and the first JPEG returned `200 image/jpeg` with length `4770189`. The gallery and usage page emitted zero console messages.
- Tony exposed three article images, no empty/numeric links, no insecure image sources, the versioned local `K-On.jpg` favicon, and zero console messages.
- The active `main.css` stylesheet contained an imported `/webjars/bootstrap/5.3.3/css/bootstrap.min.css` rule.
- After automatic deployment, the public HTTPS endpoints `/`, `/blog`, `/api/blog/v1/posts`, `/photos`, `/api/photo/v1`, `/photos/usage`, `/thebell`, `/thebell/tony`, and both pinned Bootstrap WebJar assets returned `200`. The photo API returned all 12 images and the blog API returned the configured post; equivalent POST probes remained `403`.
- The production CSP used `script-src 'self'` and no jsDelivr style source. A deployed browser pass rendered the 12-image gallery with the expected name/description alt partition, the usage warning, the configured blog post, and Tony's three images with zero warning/error console entries on every page.

## Pass / Fail

- PASS: public blog/photo APIs use the current versioned routes and standard response envelopes.
- PASS: API/WebJar public access is GET-only; mutation-shaped probes are denied.
- PASS: the blog component renders after both raw-envelope and shared-`fetchJson` payload normalization.
- PASS: unsupported tag controls and polling are removed.
- PASS: gallery configuration binding, envelope parsing, and alt fallback contracts are executable and proved against all 12 live configured records.
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
- Final single-worker `cleanTest + check`: 108 suites, 1003 Java tests, 0 failures, 3 skipped; it includes the new application-configuration binding regression.
- Final full JavaScript command `:website:jsTest`: 199 passed, 0 failed.
- Final `:website:check`: `BUILD SUCCESSFUL`; included `bootJar`, full tests, JavaScript tests, and `verifySensorRuntime`.
- Pull-request CI passed on Ubuntu, macOS, and Windows; Dependency Review and all CodeQL analyses passed. The post-merge `main` CI Build and CodeQL runs also passed for `4b82116a`.
- `dependencyInsight` selected exactly `org.webjars:bootstrap:5.3.3` on `runtimeClasspath`; the repository has no Gradle verification metadata and relies on its existing Dependency Review CI workflow plus Dependabot.
- `node --check` passed for all changed JavaScript and the new Node test; `git diff --check` passed.
- Recursive resource scans found zero Bootstrap jsDelivr references and zero The Bell `src="http://` values.

## Bugs / Follow-ups

- Fixed during local browser testing: the first implementation double-unwrapped the API response because `fetchJson` already returns `data.payload`. The regression now covers both the raw response envelope and the shared-helper payload shape.
- Fixed after independent review: `PhotoProperties` expected `photo-properties.photos` while `application.yml` supplied `photo-properties.images`. A witnessed failing configuration-context test now proves the application file binds photos, and live HTTP/browser verification proves all 12 reach the gallery.
- Fixed after the first CI rerun: the configuration regression used `@SpringBootTest`, which discovered the full application and attempted a real MongoDB connection on macOS CI. The job failed with `MongoTimeoutException`; the regression now loads the real `application.yml` through `YamlPropertySourceLoader` and binds only `PhotoProperties`, preserving the configuration contract without external services. The focused test and authoritative full suite passed locally, then all three CI platforms passed on `f5120784`.
- The first local run's unrelated startup-time OpenStreetMap catch-up listener fetched candidates and logged a duplicate-key write failure for an existing normalized restaurant name. Final gallery retesting set the supported `WFL_RESTAURANT_IMPORT_MONTHLY_ENABLED=false` boundary, so the unrelated import did not run.
- One full-check attempt overlapped a still-running invocation after a shell timeout and reported a missing Gradle binary result file. The authoritative no-overlap command used `cleanTest`, `--max-workers=1`, and disabled file watching; it passed in 2m 3s.
- The worktree continues to show the pre-existing `gradlew.bat` LF-to-CRLF checkout-only difference. It is excluded from the spoke commit.

## Publication and Closure

- PR: [#1251](https://github.com/azurras/christopherbell.dev/pull/1251), merged `2026-07-25 22:12:48 -05:00`.
- Merge commit: `4b82116a0ed489c74eed144a478f1b3a3944ada2`.
- Issues `#1131` through `#1137` closed automatically from the PR at merge.
- Automatic deployment exposed the merged behavior within approximately four minutes; live listener PID changed from `26680` to `29012`.
- Production acceptance passed over `https://www.christopherbell.dev` with no known gaps for this batch.
