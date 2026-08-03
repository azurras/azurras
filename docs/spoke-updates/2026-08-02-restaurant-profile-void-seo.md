# Restaurant Profile Void SEO

## Status

closed

## Related Work

- [Central work record](../work/2026-08-02-christopherbell-dev-restaurant-profiles-void-seo.md)
- [Implementation plan](../implementation-plans/2026-08-02-restaurant-profiles-void-seo.md)
- [Test report](../test-reports/2026-08-02-restaurant-profile-void-seo-test-report.md)

## Source Repository

- Repository: `azurras/christopherbell.dev`
- Worktree: `A:\Projects\christopherbell.dev-worktrees\restaurant-profile-void-seo`
- Branch: `codex/restaurant-profile-void-seo`
- Pull request: [#1345 Render indexable Void restaurant profiles](https://github.com/azurras/christopherbell.dev/pull/1345)
- Branch tip tested: `4535eb0ecad0711e3a509f4f37ec31230cc50d6a`
- Merged SHA: `363bb986581c4d20df3434154844807ce88701e4`

## Changes Made

- Added one immutable public-only restaurant profile page model and mapping service.
- Server-rendered complete semantic public profile content, one canonical URL, and conditional schema.org `Restaurant` JSON-LD.
- Preserved content-free `404` responses with `noindex,nofollow` and no JSON-LD for missing or malformed profiles.
- Prevented member rating/favorite state, creator/modifier identities, and audit fields from crossing the public view boundary.
- Reduced the browser module to authenticated personal-control progressive enhancement; anonymous visitors perform no redundant detail fetch.
- Moved profile presentation into scoped `whats-for-lunch.css` Void ownership with responsive, overflow-safe, keyboard-visible behavior.
- Updated WFL, JavaScript, and stylesheet ownership documentation.

## Commits

- `df77ef43` Render public restaurant profile data
- `64ee8195` Server render restaurant profiles
- `568dc3a9` Enhance restaurant member controls
- `dcd35a94` Style restaurant profiles in Void
- `7efb54b5` Document indexable restaurant profiles
- `4535eb0e` Keep profile rating signal intact

## Validation

- Regression-first focused Java, raw-view, JavaScript, CSS ownership, accessibility, desktop-wrap, and mobile-overflow tests passed.
- `:website:check --rerun-tasks --no-daemon --console=plain` passed in 3m14s with 21 tasks executed.
- JavaScript suite passed 320 tests; Windows production-script suite passed 74 tests with zero failures.
- Isolated port `8094` HTTP and browser verification passed for complete, sparse, missing, anonymous, authenticated, stale-session, desktop, mobile, and keyboard cases.
- GitHub Ubuntu, macOS, Windows, dependency-review, and CodeQL checks passed.
- Production verification passed through loopback and `https://www.christopherbell.dev` after listener rotation from PID `55848` to `59036`.

## Files Touched

The primary files are `RestaurantProfilePage.java`, `RestaurantProfilePageService.java`, `WhatsForLunchViewController.java`, `restaurant.html`, `restaurant-profile.js`, and `whats-for-lunch.css`, with focused Java/JavaScript tests and feature documentation.

## Blockers and Risks

None. `gradlew.bat` remains a line-ending-only worktree artifact and was never staged. The authoritative checkout at `A:\Projects\christopherbell.dev` was not modified.

## Next Actions

None required.
