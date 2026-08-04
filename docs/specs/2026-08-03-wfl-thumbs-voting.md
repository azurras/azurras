# What's For Lunch Thumbs Voting

## Document Status

complete

## Purpose

Replace the What's For Lunch 1–5 restaurant rating system with a binary thumbs-up/thumbs-down vote system. Convert every valid stored rating deterministically, expose only vote-based API and UI contracts, preserve useful public ranking and structured data, and keep higher-approval restaurants more likely to appear in weighted lunch selections.

## Background

The current WFL domain stores one integer rating from 1 through 5 per account and restaurant in `whatsforlunch_ratings`. Public restaurant details expose `ratingSum` and `ratingCount`; authenticated details also expose `myRating`. The picks page, restaurant profiles, Favorites, and Top 10 Rated render star-scale summaries and controls. Mongo aggregation ranks Top 10 Rated by average rating and count. Daily and shared-session restaurant selection use a confidence-adjusted 1–5 rating weight.

The user approved a complete binary replacement with these decisions:

- Existing ratings `3`, `4`, and `5` become thumbs up.
- Existing ratings `1` and `2` become thumbs down.
- Top 10 Rated becomes Top 10 Liked.
- Public summaries show approval percentage plus up/down counts.
- Lunch selection uses smoothed weighting so one early vote does not dominate.
- Old numeric rating writes are rejected immediately rather than translated.
- Persisted data receives one clean versioned migration rather than indefinite dual-schema reads.

## Goals

1. Store one `UP` or `DOWN` vote per account and restaurant.
2. Convert every valid existing 1–5 rating without losing account, restaurant, or timestamp identity.
3. Replace numeric rating request and response fields with vote-based contracts.
4. Show approval percentage, thumbs-up count, and thumbs-down count on every WFL restaurant surface.
5. Rename the public leaderboard to Top 10 Liked and rank it deterministically.
6. Use binary approval to make better-liked restaurants more frequent and disliked restaurants less frequent in both daily and shared-session picks.
7. Preserve server-rendered indexable restaurant profiles and safe structured data.
8. Deliver through regression-first implementation, local migrated-database/browser testing, PR/CI, protected production deployment, and production verification.

## Non-Goals

- No change to Favorites, shared-session participant voting, restaurant import, location resolution, daily-pick count, or nearby-search radius rules.
- No permanent compatibility support for 1–5 write payloads.
- No vote deletion or neutral vote state after a member has voted; a member may change between `UP` and `DOWN`.
- No collection rename. The existing `whatsforlunch_ratings` collection and unique restaurant/account index remain to reduce migration risk.
- No change to the approved Void visual system beyond replacing star controls and score language with thumb controls and vote summaries.

## Domain and Persistence Requirements

### Vote model

- Production code uses vote terminology, including `RestaurantVote`, vote requests, vote summaries, and vote services/repositories.
- Each document retains the existing `_id`, `restaurantId`, `accountId`, `createdOn`, and `lastUpdatedOn` values.
- The persisted binary field is `vote` with exactly `UP` or `DOWN`.
- The legacy `rating` field is absent after migration.
- The existing compound unique index on `restaurantId` plus `accountId` remains authoritative.

### V013 migration

- Add the next immutable application migration after V012.
- Before writing, validate that every document requiring conversion has a numeric integer rating from 1 through 5.
- If any legacy document is malformed, fail the migration without guessing, deleting, or partially converting data.
- Convert `3–5` to `vote: "UP"` and `1–2` to `vote: "DOWN"` in bounded, stable `_id` batches.
- Remove `rating` from each converted document.
- Treat already-converted valid vote documents as idempotently complete for retry safety.
- Reject documents that contain contradictory valid `rating` and `vote` values.
- Record a fixed migration checksum and test the migration against mixed legacy/already-converted, empty, malformed, and boundary datasets.

## API Requirements

- Add vote-only member write endpoints using request value `vote: "UP" | "DOWN"`.
- The provider-ID-safe request keeps `restaurantId` in JSON rather than requiring it in the path.
- Remove the numeric rating paths from browser API routing and do not accept numeric values on the new endpoint.
- Requests containing `rating`, numeric `vote`, unknown strings, null, or missing vote return the established stable invalid-request HTTP 400 envelope.
- Authenticated callers may change an existing vote by sending the other value.
- Sending the already-selected value is idempotent and returns current aggregate/personal state.
- Public restaurant details expose `upVotes`, `downVotes`, and `voteCount`.
- Authenticated restaurant details additionally expose `myVote`; anonymous responses omit or null personal state consistently with the existing privacy boundary.
- Legacy response fields `ratingSum`, `ratingCount`, and `myRating` are removed.
- Existing authentication, CSRF, rate limiting, and restaurant-not-found boundaries remain unchanged.

## Public Summary and Leaderboard Requirements

- The shared summary format is `83% liked · 10 up · 2 down`.
- Approval percentage is the rounded whole-number value `100 * upVotes / voteCount`.
- Zero votes display `No votes yet`; code never divides by zero or fabricates a percentage.
- Top 10 Rated becomes Top 10 Liked in visible headings, navigation, documentation, accessibility labels, canonical metadata, and sitemap membership.
- The canonical page route is `/wfl/top-liked` and the canonical API resource is `/top-liked`.
- `/wfl/top-rated` permanently redirects to `/wfl/top-liked` to preserve public links and search equity.
- The old numeric rating write endpoints do not receive compatibility translation.
- Top 10 Liked ordering is:
  1. raw thumbs-up percentage descending;
  2. total vote count descending;
  3. restaurant ID ascending.
- Restaurants with zero votes are excluded from Top 10 Liked.

## Selection Weighting Requirements

- Daily picks and shared-session restaurant selection consume the same immutable vote summary type.
- Unvoted restaurants have neutral selection weight `1.0`.
- Apply a neutral three-vote prior:

```text
adjustedApproval = (upVotes + 1.5) / (upVotes + downVotes + 3)
```

- Convert adjusted approval into the existing approved frequency range with piecewise linear interpolation:
  - adjusted `0.0` → weight `0.35`;
  - adjusted `0.5` → weight `1.0`;
  - adjusted `1.0` → weight `2.0`.
- Validate that vote counts are nonnegative and `upVotes + downVotes == voteCount`; reject malformed summaries.
- Keep weighted sampling without replacement, unique-candidate validation, bounded requested counts, and deterministic injected-random tests unchanged.

## Browser and Server-Rendered UI Requirements

- Picks, restaurant profiles, and Favorites use the same shared vote-summary formatter.
- Signed-in vote controls contain exactly two buttons: Thumbs up and Thumbs down.
- Buttons use meaningful text or accessible names, `aria-pressed`, visible focus, and the established Void control styling.
- Selecting the opposite button updates the personal vote and public aggregate without rebuilding unrelated server-rendered profile content.
- Selecting the active button is idempotent.
- Anonymous restaurant profiles remain meaningful without JavaScript and make no personal-detail request.
- Personal-control request failures remain local and never erase public profile content.
- New vote endpoints reject numeric payloads with the stable HTTP 400 envelope. Cached old assets that call removed numeric-rating routes receive HTTP 404; the new versioned assets use only the vote contract.

## Search and Structured Data Requirements

- Valid restaurant profiles remain indexable with one canonical URL and sitemap membership.
- Missing profiles remain HTTP 404 with `noindex,nofollow` and no Restaurant JSON-LD.
- For voted restaurants, schema.org `aggregateRating` uses:
  - `ratingValue`: raw whole-number approval percentage;
  - `bestRating`: `100`;
  - `worstRating`: `0`;
  - `ratingCount`: total vote count.
- Profiles with zero votes omit `aggregateRating`.
- Personal `myVote`, account identity, and audit fields never appear in public HTML or JSON-LD.

## Error and Rollback Behavior

- Migration validation failures prevent the release from becoming ready and leave the prior production release serving.
- The migration never guesses malformed legacy values or silently drops votes.
- Invalid API writes return HTTP 400; missing restaurants preserve the existing not-found behavior; unauthenticated writes preserve the existing authentication response.
- A browser vote failure renders a bounded local error and retains the last server-rendered public summary.
- The production deployment remains governed by the protected candidate-smoke, listener-switch, endpoint verification, and automatic rollback path.

## Expected Modules

- `website/src/main/java/dev/christopherbell/configuration/mongo/migration/`
- `website/src/main/java/dev/christopherbell/whatsforlunch/restaurant/model/`
- `website/src/main/java/dev/christopherbell/whatsforlunch/restaurant/rating/` or a focused renamed vote package
- `website/src/main/java/dev/christopherbell/whatsforlunch/restaurant/selection/`
- `website/src/main/java/dev/christopherbell/whatsforlunch/restaurant/RestaurantService.java`
- `website/src/main/java/dev/christopherbell/whatsforlunch/restaurant/RestaurantController.java`
- `website/src/main/java/dev/christopherbell/whatsforlunch/restaurant/session/`
- `website/src/main/java/dev/christopherbell/view/wfl/`
- `website/src/main/resources/templates/restaurant.html`
- `website/src/main/resources/static/js/lib/wfl-ui.js`
- `website/src/main/resources/static/js/whats-for-lunch.js`
- `website/src/main/resources/static/js/restaurant-profile.js`
- `website/src/main/resources/static/js/wfl-list.js`
- `website/src/main/resources/static/css/whats-for-lunch.css`
- Focused Java and JavaScript tests plus WFL/model/rating documentation.

## Validation Plan

1. Regression-first migration tests prove exact 1/2 down and 3/4/5 up conversion, preservation of identity/timestamps, retry safety, bounded iteration, and fail-closed malformed handling.
2. Model/service/controller/security tests prove vote validation, idempotent writes, vote changes, public/personal fields, authentication, and stable errors.
3. Mongo aggregation tests prove counts, percentage ordering, vote-count tie-break, stable ID tie-break, limits, and zero-vote exclusion.
4. Selector tests prove neutral prior, approved anchor weights, monotonic behavior, malformed-summary rejection, deterministic sampling, and shared daily/session use.
5. JavaScript tests prove shared summary formatting, two-button accessible controls, request payloads, mutation updates, anonymous zero-fetch behavior, and local failures.
6. Raw view tests prove profile text, canonical metadata, percentage JSON-LD, zero-vote omission, privacy, 404 noindex, and old-route redirect.
7. Full `:website:check` and repository-required JavaScript/production checks pass.
8. Run the packaged app on a non-8080 port with a disposable Mongo database containing legacy 1–5 fixtures; verify V013 data and indexes after startup.
9. Exercise complete, zero-vote, malformed-request, anonymous, authenticated up/down/change, Top 10 Liked, redirect, sitemap, desktop, mobile, keyboard, and console cases.
10. Publish a PR, pass Linux/macOS/Windows CI, dependency review, and CodeQL, merge, deploy the merged SHA, and verify public/local health, live assets, migrated aggregates, leaderboard, profiles, services, and database migration state.

## Acceptance Criteria

- No user-facing WFL star scale or “Top 10 Rated” language remains except the intentional redirect compatibility path and historical migration tests/documentation.
- Every valid legacy rating is represented by the required binary vote after V013.
- Numeric rating writes are rejected.
- Public summaries and Top 10 Liked show correct approval percentage and counts.
- Higher-approval restaurants have monotonically higher selection weights; lower-approval restaurants have lower weights; unrated remains neutral.
- All profile SEO/privacy guarantees from PR #1345 remain intact.
- Automated, alternate-port, browser, CI, merge, production, and Builder closeout evidence pass with no unresolved blocker.

## Open Questions

None. The product, migration, compatibility, UI, ranking, weighting, failure, and validation decisions are approved.
