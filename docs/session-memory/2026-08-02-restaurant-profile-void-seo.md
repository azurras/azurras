# 2026-08-02 Restaurant Profile Void SEO

## 22:47 - Completed restaurant profile Void and indexing delivery

### Request

Apply the established Void CSS to restaurant profile pages and make valid profiles indexable by search engines. Preserve the approved boundary that all valid profiles are indexable, public content is server-rendered, personal controls are progressive enhancement, and private member/audit fields never enter public HTML or structured data.

### Project Context

The Builder hub coordinated `azurras/christopherbell.dev`. Work used the isolated worktree `A:\Projects\christopherbell.dev-worktrees\restaurant-profile-void-seo` from base `9c69623049829394f245515b8d1751c9f7579271`; the dirty authoritative checkout at `A:\Projects\christopherbell.dev` was preserved. Production port `8080` remained untouched until alternate-port acceptance and PR CI passed.

### Work Completed

- Added immutable `RestaurantProfilePage` and `RestaurantProfilePageService` public projection types.
- Replaced the limited social-preview projection and server-rendered complete semantic public restaurant details in `restaurant.html`.
- Added canonical/social metadata and conditional Jackson-serialized schema.org `Restaurant` JSON-LD with safe optional-field handling.
- Reduced `restaurant-profile.js` to authenticated rating/favorite enhancement and kept anonymous/public content independent of API fetch success.
- Moved profile presentation from shared `main.css` into scoped `whats-for-lunch.css` Void ownership.
- Added Java and JavaScript coverage for hostile/sparse data, invalid IDs, privacy, raw SSR output, member controls, stylesheet ownership, accessibility, desktop wrapping, and mobile overflow.
- Updated WFL, JavaScript, and CSS ownership documentation.
- Opened PR #1345; all CI and CodeQL checks passed; squash-merged as `363bb986581c4d20df3434154844807ce88701e4`.
- Deployed through the protected Windows production path and verified the new release through loopback and Cloudflare.

### Decisions

- One public-only immutable page model is the only restaurant object exposed to Thymeleaf.
- All valid profiles remain in the public sitemap regardless of rating.
- Missing/malformed profiles remain content-free 404 responses with `noindex,nofollow` and no JSON-LD.
- Unsafe website values and unavailable optional properties are omitted.
- Top Rated and Favorites retain their prior presentation.

### Validation

- Regression-first focused tests all passed.
- Final `:website:check --rerun-tasks --no-daemon --console=plain`: `BUILD SUCCESSFUL in 3m 14s`, 21 tasks executed.
- JavaScript: 320 passed. Windows production scripts: 74 passed, zero failed.
- Candidate port `8094`: liveness/readiness, robots, sitemap, complete/sparse/missing raw HTML, parsed JSON-LD, privacy sentinels, desktop/mobile layout, keyboard focus, authenticated rating/favorite mutations, and stale-session fallback passed.
- Browser console errors: none.
- GitHub: Ubuntu, macOS, Windows, dependency review, Actions CodeQL, Java/Kotlin CodeQL, and JavaScript/TypeScript CodeQL passed.
- Production: listener rotated from PID `55848` to PID `59036`; local and public liveness/readiness returned 200; sitemap contained 7,340 profile URLs; real profile `Boogaloos` returned canonical indexable Void HTML and Restaurant JSON-LD; versioned assets `/29e5fee580df3dc0a71e/css/whats-for-lunch.css` and `/29e5fee580df3dc0a71e/js/restaurant-profile.js` returned exact new behavior; missing profile returned 404 noindex without JSON-LD; all required services ran.
- Candidate PID and exact isolated database `christopherbell_dev_restaurant_profiles_void_seo` were removed after local testing.

### Current State

PR #1345 is merged and production is healthy. The spoke worktree still has only the known line-ending-only `gradlew.bat` modification, which was never staged. The Builder test report is `docs/test-reports/2026-08-02-restaurant-profile-void-seo-test-report.md`.

### Follow-ups

None required. Preserve the public/private projection invariant in future restaurant profile work.
