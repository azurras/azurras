# christopherbell.dev Restaurant Profiles Void and Search Indexing

## Document Status

ready-for-execution

## Purpose

Give every valid public restaurant profile a first-class Void presentation and make its useful restaurant information directly indexable by search engines, while keeping member-specific rating and favorite data private.

## Background

The public route `/wfl/restaurants/{restaurantId}` already resolves valid restaurants, emits a canonical URL and social metadata, returns a content-free `404` with `noindex,nofollow` for missing profiles, and is represented in the public sitemap. However, the profile body is currently an empty client-side mount populated by JavaScript. Search crawlers that do not execute the application JavaScript receive only the page shell and limited metadata rather than the address, aggregate rating, contact information, source details, and actions visible to a browser user.

The profile also remains on the light site shell and does not load the scoped What's for Lunch Void stylesheet that now owns the Decision Console. The user approved applying the established Void visual system to restaurant profiles without redesigning the separate Top Rated or Favorites pages.

## User Decisions

1. Every valid public restaurant profile should be indexable, not only selected or highly rated restaurants.
2. Use server-side rendering with progressive enhancement rather than a duplicated crawler-only snapshot or metadata-only optimization.
3. Build one public-only page model from the canonical restaurant detail.
4. Render the complete public profile in Thymeleaf before JavaScript runs.
5. Use JavaScript only to add signed-in personal rating and favorite controls.
6. Extend the approved What's for Lunch Void language to profile pages while preserving their single-restaurant information hierarchy.

## Goals

1. Return meaningful restaurant content in the raw HTML for every valid profile.
2. Provide unique, safe search metadata and structured restaurant data.
3. Preserve canonical URLs, sitemap discovery, and clean `404`/`noindex` behavior.
4. Prevent personal member state and audit data from entering public HTML or structured data.
5. Give restaurant profiles a responsive, accessible Void layout consistent with the WFL Decision Console.
6. Keep the public profile usable if JavaScript is unavailable or the member enhancement request fails.

## Non-Goals

- Do not make Favorites public or indexable.
- Do not redesign the Top Rated or Favorites pages.
- Do not expose `myRating`, `myFavorite`, creator identity, modification identity, or audit timestamps in public HTML or JSON-LD.
- Do not create a separate crawler route, hidden keyword content, or duplicated SEO-only profile document.
- Do not change restaurant eligibility, import behavior, rating rules, profile URL shape, database schema, or WFL selection behavior.
- Do not make missing or malformed restaurant IDs indexable.

## Public Profile Model

Create one immutable public-only page model from the canonical `RestaurantDetail` result. It owns only information appropriate for an anonymous visitor and search crawler:

- restaurant ID;
- name;
- cuisine when present;
- formatted postal address;
- aggregate rating value and count when ratings exist;
- telephone when present;
- safe external website URL when present;
- source/type label when appropriate for public display;
- directions URL;
- canonical URL;
- page title and description; and
- structured-data fields, including coordinates when valid.

The model must not contain member-personalized or audit fields. Keeping those values out of the model is the primary privacy boundary; templates should not receive data they are forbidden to render.

## Server Rendering and Progressive Enhancement

The restaurant controller loads the canonical detail once, creates the public page model, and gives Thymeleaf everything required to render the visible profile. The raw response must include the restaurant name, cuisine/location context, address, aggregate rating when available, contact/action details when available, and source information without depending on JavaScript.

JavaScript enhances a dedicated member-controls mount only for an authenticated visitor. It may fetch the existing detail endpoint to add personal rating and favorite controls. Anonymous visitors must not incur an unnecessary profile-detail fetch merely to recreate content already present in HTML.

If member enhancement fails, retain the complete public profile and show a concise inline status in the member-controls area. An API failure must never replace or clear the public content.

## Search Indexing and Structured Data

### Valid Profiles

- Return `200` with meaningful public HTML.
- Emit a unique title and useful description derived from safe restaurant fields.
- Emit one canonical URL using the encoded canonical restaurant ID.
- Do not emit `noindex`.
- Remain discoverable through the public sitemap.
- Be reachable through ordinary profile links from WFL surfaces.

### JSON-LD

Emit one valid `application/ld+json` object using `https://schema.org/Restaurant`. Include only fields that exist and pass the application's safety rules:

- `name`;
- `servesCuisine`;
- canonical `url`;
- `PostalAddress` components;
- `GeoCoordinates` when latitude and longitude are valid;
- `telephone` when present;
- the restaurant's safe external website as `sameAs` when present; and
- `AggregateRating` only when the restaurant has one or more ratings and the value/count are valid.

Structured data must be serialized safely rather than assembled through unescaped string concatenation. It must contain the same public facts shown on the page and must never include member state or audit fields.

### Missing or Malformed Profiles

- Return a content-free `404`.
- Emit `noindex,nofollow`.
- Do not emit restaurant JSON-LD.
- Do not provide a misleading canonical URL for a nonexistent restaurant.

`robots.txt` must continue to permit public restaurant profiles and advertise the sitemap. Favorites remains private and non-indexable; Top Rated and the restaurant sitemap remain public.

## Void Profile Design

### Ownership and Scope

Extend `whats-for-lunch.css` so its scoped Void rules own both the chooser and restaurant profiles. The profile page opts into `void-shell-page` and `lunch-void-page` and loads the WFL stylesheet after `main.css`. All new rules remain scoped beneath the WFL Void page class to avoid affecting unrelated pages.

### Information Hierarchy

- Use a compact Void hero with a gold `Restaurant signal` label, restaurant name as the single page H1, and cuisine/location context.
- Present the aggregate public rating as a clear signal panel without implying that the profile is selected, sponsored, or objectively ranked.
- Use a structured detail surface for address, contact, source, website, and directions.
- Keep external actions conventional and understandable, with safe-link behavior preserved.
- Reserve a clearly labeled member area for personal rating and favorite controls when signed in.

### Visual Language

- Reuse the WFL near-black layered background, subtle grid, teal signal accents, restrained gold highlights, panel borders, and typography hierarchy.
- Use a purpose-built single-restaurant layout rather than copying the chooser's three-card grid.
- Use a balanced two-column detail layout on wider screens and a single stacked column on narrow/mobile screens.
- Preserve sufficient contrast, visible `:focus-visible` states, semantic landmarks/headings, and keyboard operation.
- Do not encode rating or state using color alone.
- Respect `prefers-reduced-motion` for any new transition or animation.

## Expected Files and Modules

- `website/src/main/java/dev/christopherbell/view/wfl/WhatsForLunchViewController.java`
- `website/src/main/java/dev/christopherbell/view/wfl/RestaurantSocialPreviewService.java` or a focused public-profile view-model service beside it
- a focused immutable restaurant public-profile/page model under the WFL view package
- `website/src/main/resources/templates/restaurant.html`
- `website/src/main/resources/static/js/restaurant-profile.js`
- `website/src/main/resources/static/css/whats-for-lunch.css`
- relevant controller, view-model/service, template, JavaScript, stylesheet, sitemap, and robots tests

The implementation plan must confirm exact ownership and literal edit ranges from the refreshed isolated worktree before execution.

## Error and Edge-Case Behavior

- A valid profile without cuisine, phone, website, source metadata, coordinates, or ratings still renders a useful indexable page; absent optional fields are omitted cleanly.
- An unrated restaurant must not emit a fabricated aggregate rating.
- Invalid external website schemes remain non-clickable and must not enter JSON-LD.
- Missing address components are formatted without placeholder punctuation or invented locations.
- Member endpoint `401` or anonymous state does not create an error banner and does not fetch when authentication status is already known server-side.
- Other member enhancement failures produce a local inline status while preserving public content.
- The public page must remain understandable and navigable with scripts disabled.

## Validation Plan

### Server, Search, and Template Tests

1. Verify a valid profile response contains its public name, address, rating/contact data when present, canonical URL, and no `noindex` before JavaScript executes.
2. Verify unique safe title and description generation for complete and sparse records.
3. Verify JSON-LD parses as JSON, uses `Restaurant`, and conditionally includes address, coordinates, telephone, external website, cuisine, and aggregate rating.
4. Verify hostile or malformed restaurant fields cannot break HTML, metadata, or JSON-LD contexts.
5. Verify member state and audit fields never appear in public HTML or JSON-LD.
6. Verify missing and malformed IDs return content-free `404` responses with `noindex,nofollow` and no restaurant structured data.
7. Verify encoded canonical URLs, restaurant sitemap membership, and public `robots.txt` sitemap/disallow behavior.

### JavaScript and CSS Tests

1. Verify anonymous pages keep server-rendered content and perform no redundant detail fetch.
2. Verify authenticated enhancement adds personal rating and favorite behavior without duplicating public fields.
3. Verify enhancement failure retains public content and shows only the inline member-area status.
4. Verify the template opts into the Void shell and the WFL stylesheet.
5. Verify CSS covers hero, rating signal, detail grid, actions, member controls, focus, error, responsive, and reduced-motion states while remaining page-scoped.

### Full and Runtime Verification

1. Run focused Java/service/controller/template/JavaScript/static tests, then the full `:website:test` and `:website:check` gates with a task-private Gradle home.
2. Package the candidate and run it on a non-8080 port with an isolated MongoDB database.
3. Inspect raw HTTP for a complete valid profile, sparse/unrated profile, encoded-ID profile, missing profile, `robots.txt`, and sitemap.
4. Validate JSON-LD with a parser and verify no personal/audit fields are present.
5. Exercise anonymous and authenticated profiles in the browser at desktop and mobile widths, including keyboard focus, website/directions, personal rating, favorite, scripts-disabled content, and member API failure fallback.
6. Confirm the production listener on port 8080 remains untouched during candidate validation.
7. Publish a PR, pass required CI, merge, deploy the exact SHA through the protected Windows workflow, and verify production identity, readiness, liveness, MongoDB health, raw indexable HTML, sitemap/robots behavior, and authenticated desktop/mobile profile presentation.

## Acceptance Criteria

- Every valid restaurant profile returns its meaningful public content in raw HTML.
- Valid profiles have unique safe metadata, one canonical URL, valid conditional Restaurant JSON-LD, sitemap discovery, and no `noindex` directive.
- Missing/malformed profiles remain content-free `404` responses with `noindex,nofollow` and no restaurant structured data.
- Public output contains no personal rating/favorite state or audit fields.
- Anonymous visitors do not need a detail API fetch to see the profile.
- Authenticated personal controls enhance the server-rendered profile without replacing it, and failures degrade locally.
- The page matches the approved scoped Void profile direction on desktop and mobile while meeting keyboard, contrast, semantic, and reduced-motion requirements.
- Focused, full, alternate-port runtime, PR CI, deployment, and production checks pass.

## Risks and Mitigations

- **Private data leakage:** reusing the API detail object in the template could expose member state. Mitigation: use an immutable public-only model that cannot carry those fields.
- **HTML/JSON-LD injection:** dynamic fields cross multiple output contexts. Mitigation: use Thymeleaf escaping and a trusted JSON serializer with hostile-input regression tests.
- **Duplicate data drift:** client and server rendering could disagree. Mitigation: server rendering owns all public fields; JavaScript owns only member-specific controls.
- **Crawler ambiguity:** partial success or missing records could be indexed incorrectly. Mitigation: retain explicit `404` plus `noindex,nofollow` behavior and test raw responses.
- **Broad CSS regressions:** shared styles could affect other pages. Mitigation: scope every addition beneath the WFL Void profile classes and leave Top Rated/Favorites unchanged.
- **Visual regressions for sparse records:** optional fields could create empty panels. Mitigation: conditionally render fields and verify complete, sparse, and unrated fixtures at desktop/mobile widths.

## Rollback and Recovery

- Code rollback: redeploy the previous known-good merged SHA.
- The feature requires no database migration or data rewrite.
- Existing profile URLs, sitemap entries, restaurant data, ratings, and favorites remain compatible with both versions.
- Reverting the controller/template/model/asset changes restores the previous client-rendered profile without data recovery.

## Open Questions

None. The user approved indexing all valid profiles, server rendering with progressive enhancement, the public-only data boundary, structured-data requirements, missing-profile behavior, and the scoped Void profile direction.
