# christopherbell.dev What's for Lunch Rating-Weighted Void Upgrade

## Document Status

ready-for-execution

## Purpose

Make the three-restaurant What's for Lunch selection rating-aware without turning it into a deterministic leaderboard, and upgrade the WFL page to the approved Void Decision Console visual language while retaining every existing user capability.

## Background

Current `azurras/christopherbell.dev` `origin/main` selects restaurants by copying the eligible candidate list, calling `Collections.shuffle`, and taking the first three. The same unweighted order helper serves:

- persisted daily lunch picks;
- on-demand browser-location and ZIP-code picks; and
- a replacement when an administrator deletes a restaurant in today's picks.

The application already stores one whole-number 1–5 rating per account and restaurant, exposes public rating counts and sums, and owns a MongoDB aggregate query repository for rating summaries. Ratings currently appear in the UI but do not affect selection probability.

The WFL page currently uses the light `site-page` shell, a photo-backed hero, Bootstrap-oriented control surfaces, and dedicated lunch styles. The site already has a mature Void shell with near-black layered backgrounds, teal signal accents, restrained gold highlights, accessible focus states, and responsive card layouts. During visual brainstorming, the user selected the **Decision Console** direction rather than a light reskin or a dense terminal layout.

## User Decisions

1. Apply weighting to every three-pick flow.
2. Treat unrated restaurants as neutral so new restaurants remain discoverable.
3. Adjust sparse rating evidence toward neutral; a single vote must not receive full influence.
4. Use moderate rather than mild or strong probability differences.
5. Use confidence-adjusted weighted sampling without replacement.
6. Upgrade the WFL page to the Void Decision Console treatment while preserving its existing functions.

## Goals

1. Make higher-rated eligible restaurants appear more often over repeated draws than neutral or lower-rated restaurants.
2. Make lower-rated eligible restaurants appear less often than neutral restaurants without setting their probability to zero.
3. Return at most three unique restaurants and preserve the current eligibility boundary.
4. Apply one selection policy consistently to daily, nearby/ZIP, and replacement draws.
5. Prevent a single rating from dominating a restaurant's odds.
6. Keep rating aggregation bounded to one query per candidate draw rather than one query per restaurant.
7. Give WFL a first-class Void identity with a compact decision console and three equal result cards.
8. Preserve accessibility, responsiveness, session stability, and all existing restaurant actions.

## Non-Goals

- Do not sort picks as a leaderboard or guarantee that the highest-rated restaurant appears.
- Do not permanently exclude low-rated or unrated restaurants.
- Do not use the current viewer's personal rating as a separate selection signal.
- Do not add paid placement, cuisine popularity, distance weighting, favorites weighting, or time-decay behavior.
- Do not change restaurant eligibility, supported metro rectangles, radius handling, cuisine filters, import behavior, rating validation, public API shapes, or database schemas.
- Do not recalculate already-persisted daily picks or active shared-session picks merely because a rating changes or a deployment occurs.
- Do not make the three displayed positions imply a quality ranking.
- Do not convert WFL into a dense administrative terminal or remove conventional labels needed for usability.

## Rating-Weighted Selection Requirements

### Candidate Eligibility

The existing service remains the sole owner of eligibility. Rating weighting runs only after the current rules have selected candidates:

- supported city/state for daily picks;
- valid restaurant ID;
- valid coordinates for nearby/ZIP picks;
- distance within the selected radius;
- matching explicit or saved cuisine filters; and
- exclusion of already-selected IDs during a replacement draw.

Ratings change probability only. They cannot make an otherwise ineligible restaurant eligible.

### Confidence-Adjusted Rating

For each candidate, calculate an adjusted rating from aggregate rating sum and count:

```text
adjustedRating = (ratingSum + 3.0 * 3) / (ratingCount + 3)
```

The prior represents three neutral virtual ratings at 3 stars.

- An unrated restaurant has `adjustedRating = 3.0`.
- One 5-star rating produces `3.5`, not `5.0`.
- One 1-star rating produces `2.5`, not `1.0`.
- Repeated consistent ratings gradually approach their observed average.

Rating count and sum must be non-negative and the calculated score must be clamped to `[1.0, 5.0]` as defense in depth. Normal application writes continue to enforce integer ratings from 1 through 5.

### Moderate Weight Curve

Map adjusted ratings through these probability-weight anchors, linearly interpolating fractional scores:

| Adjusted rating | Selection weight |
| ---: | ---: |
| 1.0 | 0.35 |
| 2.0 | 0.60 |
| 3.0 | 1.00 |
| 4.0 | 1.50 |
| 5.0 | 2.00 |

Every weight remains positive. With sufficient evidence, a 5-star restaurant approaches twice the selection chance of an unrated/neutral restaurant, while a 1-star restaurant approaches 35% of neutral. Sparse evidence remains materially closer to neutral.

### Unique Weighted Draw

Select without replacement:

1. Sum the positive weights of the remaining candidates.
2. Draw one random value inside that total.
3. Select the candidate whose cumulative interval owns the value.
4. Remove the selected candidate.
5. Repeat until the requested count is reached or no candidates remain.

The selector must return no duplicate restaurant IDs, preserve the requested maximum, handle fewer candidates than requested, and expose deterministic randomness at a test boundary. Its production entry point uses `ThreadLocalRandom.current().nextDouble()`, while a package-private overload accepts a `DoubleSupplier` for deterministic tests. Tests must not rely on wall-clock or nondeterministic statistical outcomes.

### Rating Summary Query

Extend the aggregate rating query repository with a method that accepts the eligible candidate IDs and returns one `RestaurantRatingSummary` per rated restaurant. The query must:

- match only the supplied candidate IDs;
- group by restaurant ID;
- return rating count and sum;
- return no synthetic record for unrated restaurants; and
- perform one aggregate query for the draw.

The service maps absent summaries to the neutral prior. A rating query failure remains a visible service failure rather than silently changing product behavior back to uniform randomness.

### Selection Lifecycle

- **Nearby/browser location:** every request uses current aggregate ratings.
- **ZIP:** every request uses current aggregate ratings after resolving the ZIP origin.
- **Daily:** the next daily generation uses ratings current at generation time; the persisted list stays stable for that date.
- **Deleted-pick replacement:** only the replacement slot is drawn from the remaining eligible candidates using current ratings; surviving picks retain their order and identity.
- **Shared session:** existing session picks remain stable. Host-triggered fresh picks use the normal weighted nearby/ZIP path.

## Component Design

### `RatingWeightedRestaurantSelector`

A new small, stateless selection-policy component owns:

- neutral-prior adjustment;
- score clamping;
- piecewise-linear weight interpolation;
- weighted sampling without replacement; and
- random-boundary validation.

It receives already-eligible `Restaurant` candidates and rating summaries. It does not query MongoDB, inspect authentication, calculate distance, interpret filters, or persist picks.

### `RestaurantRatingQueryRepository`

The existing aggregation boundary gains a candidate-summary method. It remains responsible only for MongoDB aggregation and result mapping.

### `RestaurantService`

The service gains one orchestration helper that:

1. accepts eligible candidates and a requested count;
2. loads rating summaries once;
3. indexes summaries by restaurant ID; and
4. delegates to the selector.

The helper replaces the uniform shuffle in daily generation, nearby/ZIP selection, and deleted-pick replacement. Unrelated rating display, top-rated listing, import, filtering, session, and CRUD behavior stays unchanged.

## Void Decision Console Requirements

### Page Shell

- Add the established `void-shell-page` class and a WFL-specific Void page class.
- Replace the photo-backed lunch hero and white content panel with a near-black layered background, subtle grid, teal signal lines, and restrained gold accents drawn from existing Void tokens.
- Keep the global navigation and footer behavior intact.

### Information Hierarchy

- Use a compact WFL hero with the existing page title, a Decision Console eyebrow, and a concise lunch-decision description.
- Render secondary WFL navigation and freshness information as quiet Void utility surfaces.
- Style the toolbar plus active Filters, Location, or Lunch with Friends panel as one coherent decision-control deck.
- Make the refresh action visually primary while preserving its current accessible label and disabled semantics.
- Add a short note: ratings influence the draw, but every eligible restaurant remains possible.

### Pick Cards

- Display the three selections as equal-width cards on desktop and a single column on narrower screens.
- Retain `01`, `02`, and `03` as neutral identifiers only.
- Do not add labels such as “best,” “strong signal,” or ranking scores.
- Preserve restaurant name, cuisine, address, public/personal rating display, favorites, voting, directions, website, phone, and administrator deletion.
- Preserve safe external-link behavior and existing profile links.

### States and Accessibility

- Style loading, location prompt, empty, error, active session, archived session, form, success, and disabled states in the same Void language.
- Preserve semantic headings, landmarks, form labels, live regions, button states, and keyboard operation.
- Provide visible `:focus-visible` treatment with sufficient contrast.
- Do not encode rating or state using color alone.
- Retain mobile behavior and make the card/control layouts usable at the current responsive breakpoints.
- Respect `prefers-reduced-motion` for any newly added transitions or animations.

## Expected Files and Modules

- `website/src/main/java/dev/christopherbell/whatsforlunch/restaurant/RestaurantService.java`
- `website/src/main/java/dev/christopherbell/whatsforlunch/restaurant/rating/RestaurantRatingQueryRepository.java`
- new focused selector under `website/src/main/java/dev/christopherbell/whatsforlunch/restaurant/selection/`
- selector and repository/service tests under the matching `website/src/test/java` packages
- `website/src/main/resources/templates/whatsforlunch.html`
- `website/src/main/resources/static/js/whats-for-lunch.js` for the approved probability disclosure and Decision Console semantic class hooks
- `website/src/main/resources/static/css/main.css`
- relevant WFL and frontend documentation/tests

## Failure and Edge-Case Behavior

- Null/empty candidate input returns an empty selection without a rating query.
- Fewer than three candidates returns every eligible candidate exactly once.
- Duplicate candidate IDs are rejected explicitly before weighting; the final selection cannot duplicate an ID.
- Missing rating summaries are neutral.
- A present rating summary must have a positive count and a sum within `[count, 5 * count]`; corrupt summaries fail explicitly instead of influencing a draw. Valid adjusted scores are still clamped to `[1.0, 5.0]` as defense in depth.
- A random source value outside `[0.0, 1.0)` fails explicitly in the selector test boundary.
- MongoDB aggregation failures propagate through the existing API error handling and UI error state.
- Rating changes do not mutate existing daily/session pick documents.
- The Void redesign must not hide actions when optional restaurant fields are absent.

## Validation Plan

### Automated Behavior Tests

1. Prove the old uniform path fails a regression requiring rating-aware selection.
2. Verify exact adjusted-rating calculations for unrated, one 5-star, one 1-star, and established averages.
3. Verify exact weight-anchor and interpolation values.
4. Verify three unique results, fewer-than-three behavior, and no-candidate behavior.
5. Verify deterministic boundary draws select the expected weighted interval and remove the selected candidate.
6. Run a fixed-seed, high-sample deterministic distribution test proving high-rated frequency > neutral frequency > low-rated frequency and moderate ratios remain within reviewed bounds.
7. Verify rating summaries are queried once per draw for only eligible IDs.
8. Verify daily generation, nearby/browser location, ZIP, and replacement all use the weighted selector.
9. Verify persisted daily/session picks remain stable until their established refresh action.
10. Verify rating display and top-rated listing behavior remain unchanged.

### Frontend and Static Tests

1. Verify the WFL template opts into the Void shell and approved page structure.
2. Verify all existing interactive selectors and event bindings remain present.
3. Verify the probability disclosure copy is rendered safely.
4. Verify CSS covers the decision console, pick cards, controls, loading, empty, error, focus, disabled, responsive, and reduced-motion states.
5. Run established JavaScript/static asset tests and `git diff --check`.

### Full and Runtime Verification

1. Run focused selector, repository, service, controller, and WFL frontend tests.
2. Run full `:website:test` and `:website:check` with a task-private Gradle home.
3. Package the JAR and start it on a non-8080 port with an isolated MongoDB database and deterministic candidate/rating fixtures.
4. Exercise browser-location and ZIP selection, cuisine/radius filters, “Try 3 more,” rating controls, favorite/vote/session behavior, loading/error behavior, and safe links.
5. Capture desktop and narrow/mobile visual evidence for the Decision Console and keyboard-focus states.
6. Prove repeated deterministic runtime draws favor high-rated over neutral over low-rated candidates while returning unique triples.
7. Confirm the production listener on port 8080 remains untouched during candidate testing, then clean up the candidate process and isolated database.
8. Publish a PR, pass required CI, merge, deploy the exact SHA through the protected Windows workflow, and verify production commit identity, readiness, liveness, MongoDB health, and public WFL behavior.

## Acceptance Criteria

- Every new three-pick draw uses the approved confidence-adjusted weighted policy.
- Unrated restaurants have neutral weight exactly `1.0`.
- Well-established high ratings increase draw frequency and well-established low ratings reduce it according to the reviewed anchors.
- No restaurant has zero probability solely because of its rating.
- Every response contains no more than three unique eligible restaurants.
- Rating aggregation performs one candidate-bounded query per draw rather than per restaurant.
- Existing daily and session picks remain stable.
- The WFL page matches the approved Void Decision Console direction on desktop and mobile.
- All existing controls and actions remain usable by pointer and keyboard.
- Automated, alternate-port runtime, PR CI, deployment, and production checks pass.

## Risks and Mitigations

- **Popularity feedback loop:** higher exposure may create more ratings. Mitigation: cap the maximum at 2× neutral and retain positive probability for every eligible restaurant.
- **Sparse-vote gaming:** one rating could distort odds. Mitigation: use three neutral virtual ratings and the existing one-rating-per-account/restaurant ownership rule.
- **Query growth:** daily selection can contain thousands of candidates. Mitigation: use one aggregate query and review explain/runtime behavior; do not add N+1 access.
- **Statistical test flakiness:** nondeterministic tests can fail spuriously. Mitigation: inject/fix the random sequence and seed every distribution test.
- **Visual regression:** a broad dark-theme override can break nested controls. Mitigation: scope styles to the WFL Void page, exercise every state, and inspect desktop/mobile browser evidence.
- **Implied ranking:** visual emphasis could misrepresent the random picks. Mitigation: use equal cards and neutral numeric identifiers.

## Rollback and Recovery

- Code rollback: redeploy the previous known-good merged SHA.
- Selection rollback: the change introduces no schema migration; restoring the prior service and CSS returns uniform random behavior.
- Data rollback is not expected because the feature reads existing ratings and does not rewrite rating or restaurant records.
- Existing persisted daily/session pick documents remain compatible with both versions.

## Open Questions

None. The user approved the weighting scope, neutral handling, confidence adjustment, moderate strength, weighted-sampling approach, Decision Console direction, component boundaries, lifecycle, and validation requirements.
