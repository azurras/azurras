# christopherbell.dev Performance, Scalability, and Library Optimization

## Document Status

Ready for review.

## Purpose

Improve the runtime speed, resource bounds, and horizontal-scaling readiness of
`azurras/christopherbell.dev` while consolidating genuinely shared Java behavior
into `cbell-lib` and genuinely shared browser behavior into `static/js/lib`.

The work uses a hot-path-first strategy: measure current behavior, remove
avoidable work from common requests, bound growth and concurrency, and extract
only stable abstractions with multiple consumers.

## Background

The website is a Java 25 and Spring Boot 4.1 monolith backed by MongoDB, with
Thymeleaf pages and browser-native JavaScript modules. The design review used a
clean current-mainline checkout at
`A:\Projects\christopherbell.dev-worktrees\repository-documentation-coverage-20260729`
on `origin/main` commit `5de2a8b02941ff7e95b6f2648b7bada9397f68b9`.
The dirty authoritative checkout at `A:\Projects\christopherbell.dev` was not
modified.

The current source contains approximately:

- 570 website Java files and 42,828 source lines;
- 58 browser JavaScript files and 12,835 source lines; and
- 31 `cbell-lib` Java files and 1,347 source lines.

The global `app.js` graph currently reaches 14 modules totaling 172,868 raw
bytes and 3,956 source lines. `main.css` is 154,946 raw bytes and mixes global
styles with feature-specific Command Center, Shared Folder, Void, and media
player styles. Production already serves release-versioned static assets with
immutable caching and Cloudflare Brotli compression, so the remaining browser
opportunity is primarily unnecessary parsing, execution, CSS matching, and
whole-asset invalidation after unrelated backend commits.

The active Builder campaign for GitHub issues #1258-#1307 already owns many
data-growth and concurrency improvements. This specification reuses those
issues instead of duplicating them, especially #1273-#1279, #1281-#1287, and
#1290-#1297.

## Validated Optimization Opportunities

### Authentication and request overhead

- `JwtAuthenticationFilter.shouldNotFilter` skips a public request only when no
  bearer token and no browser cookie are present. Logged-in browsers therefore
  authenticate public static asset requests even though those assets cannot use
  the resulting security context.
- Browser-cookie authentication loads the browser session and then separately
  loads the account. Interactive requests can additionally save the full
  session to advance idle state.
- A newly versioned page can request the global JavaScript graph, page script,
  stylesheet, and other assets; authenticating each network-served static asset
  multiplies otherwise unnecessary MongoDB reads.

### Query amplification and unbounded state

- `ConversationService.getConversations` batches account hydration but calls
  `countByRecipientAccountIdAndSenderAccountIdAndReadFalse` once for every
  returned conversation.
- `VehicleVinDecodeRateLimiter` stores one Bucket4j bucket per client key in an
  unbounded `ConcurrentHashMap`, despite the global rate-limit store already
  having bounded expiry behavior.
- Several synchronous HTTP clients convert complete upstream responses to
  strings without a byte limit. Timeouts exist, but an oversized successful
  response can still occupy avoidable heap.
- Scheduled writers use multiple coordination patterns. The reusable Mongo
  lease infrastructure already supports several features, but remaining
  writers must be classified as leased, idempotent duplicate-safe, or
  single-instance-only.

### Shared-code boundaries

- `StableCursor` and `StableCursorCodec` are used by messages, notifications,
  feeds, discovery, and federation but remain in the website module.
- The Mongo lease primitives are used by Canes collection, vehicle enrichment,
  random-VIN import, WFL import, Music metadata, Music radio, and migration
  coordination but remain in website configuration code.
- `TestUtil` is compiled into the production `cbell-lib` artifact even though
  every website consumer is test code.
- JJWT dependencies are declared by `cbell-lib`, while the only production JWT
  consumer is website `PermissionService`.
- The generic workflow engine has only one current production consumer, WFL,
  and therefore does not satisfy the repository rule for shared-library code.
- Browser code repeats WFL navigation and formatting, alert/status rendering,
  URL-punctuation trimming, and HTML escaping across multiple modules.

## Goals

- Remove authentication database work from static asset requests.
- Reduce browser-session database round trips and coalesce idle-state writes
  without weakening account suspension, credential-change invalidation, role
  changes, logout, or session revocation.
- Keep Mongo query groups constant as page size grows for optimized endpoints.
- Bound every reviewed collection read, process-local map, outbound response,
  and scheduled-work claim.
- Reduce unrelated JavaScript parsed on simple pages by at least 50 percent from
  the measured global graph.
- Stop unrelated pages from loading feature-exclusive CSS.
- Preserve immutable caching while allowing backend-only releases to reuse
  unchanged asset URLs.
- Move stable, multi-consumer Java behavior to `cbell-lib`.
- Move stable, multi-consumer browser behavior to `static/js/lib`.
- Preserve feature ownership and avoid abstractions created solely to shorten a
  large file.
- Reuse existing GitHub issues and the active 50-issue campaign where scope
  overlaps.

## Non-Goals

- Do not introduce a frontend framework, npm workflow, transpiler, bundler, or
  new browser build pipeline.
- Do not introduce Redis, a message broker, or reactive server rewrite before
  measured demand justifies that infrastructure.
- Do not move Shared Folder, WFL, Music, federation, vehicle, or post business
  rules into a shared library.
- Do not treat file size alone as a runtime optimization.
- Do not weaken authentication freshness, authorization, CSRF protection,
  session invalidation, outbound-network policy, or audit behavior.
- Do not duplicate issues already tracked by #1258-#1307.
- Do not modify or clean the dirty authoritative spoke checkout.
- Do not treat design or implementation-plan approval as authorization to edit,
  publish, merge, or deploy spoke code; execution requires separate explicit
  user authorization.

## Performance Architecture

Work proceeds through four ordered lanes:

1. Measure current request latency, Mongo query counts, browser dependency
   sizes, scheduled-job behavior, and bounded-store size.
2. Remove avoidable work from common requests and browser startup.
3. Bound data growth, upstream work, local coordination, and multi-instance
   execution.
4. Extract stable behavior only after the shared contract and consumers are
   explicit.

The following invariants apply throughout:

- static asset requests perform zero authentication-related MongoDB queries;
- optimized paged responses do not add one repository call per returned item;
- optional frontend modules fail without breaking core navigation or content;
- bounded stores define both expiry and maximum cardinality;
- lease loss stops further writes;
- oversized upstream responses fail with a safe typed outcome; and
- every performance claim has before-and-after evidence.

## Proposed Backend Approach

### 1. Static-resource authentication bypass

Make the authentication filter skip recognized static resource paths regardless
of bearer or cookie presence. The bypass applies only to already-public static
assets and must not include protected media, private downloads, service-worker
authorization exchanges, or API routes.

Add tests proving that:

- authenticated and unauthenticated static requests bypass session and account
  repositories;
- protected APIs still authenticate;
- public APIs that tailor results to an authenticated viewer still authenticate
  when credentials are supplied; and
- static cache and security headers remain unchanged.

### 2. Browser-session operation reduction

Reduce cookie authentication to one MongoDB round trip by making the session
record the authoritative authentication snapshot needed for request setup.
Centralize session revocation for account suspension, role changes, password or
credential changes, account deletion, and other security-fingerprint changes.

Coalesce idle-session persistence with an atomic conditional update. A normal
interactive request must not rewrite the full session when the last durable
touch is within the configured coalescing interval. Token rotation remains an
explicit atomic operation with its current overlap behavior.

Backward compatibility must be explicit: sessions missing newly required
snapshot fields are rejected or safely upgraded; they must never receive an
assumed role or active state. Concurrency tests must cover authentication racing
with revocation and rotation.

### 3. Conversation unread-count batching

Replace the per-conversation unread count call with one aggregation or one
bounded batch query keyed by the current user and returned conversation IDs.
The endpoint must retain its current ordering, archive visibility, limit cap,
display names, and unread semantics.

Tests must compare query counts for one and fifty conversations and prove the
count remains constant.

### 4. VIN limiter and outbound-response bounds

Replace the VIN decoder's unbounded bucket map with a maximum-size,
inactivity-expiring store. Preserve per-client capacity and batch-token costs.
Expose store size and eviction behavior to tests and metrics.

Add byte-bounded response handling before string or JSON materialization for the
NHTSA, OpenStreetMap, Random VIN, robots, and Canes clients. Each client keeps
feature-specific timeouts, status handling, content validation, and retry or
cooldown policy. Slow public upstream calls must also have an explicit
concurrency bulkhead so upstream latency cannot consume an unbounded number of
request threads.

### 5. Scheduled-work classification

Inventory every `@Scheduled` writer and classify it as:

- coordinated through the shared Mongo lease;
- atomically claimed per work item;
- idempotent and safe to duplicate; or
- explicitly unsupported in a multi-instance deployment.

Move eligible jobs to the shared lease contract. Tests must cover contention,
renewal, ownership loss, and retry after expiry.

### 6. Existing scaling issues

Do not create replacement issues for existing work. Execute the current issue
contracts in dependency order, with particular focus on:

- post relationship and feed scaling: #1273-#1279;
- WFL membership, concurrency, retention, hydration, inventory, and aggregation:
  #1281-#1287; and
- Shared Folder catalog, pagination, concurrency, and retention: #1290-#1297.

## Proposed Browser Approach

### 1. Conditional global modules

Keep only navigation, footer, authentication events, and minimal media-routing
behavior in the initial `app.js` graph. Load blog and gallery components only
when their containers exist. Load the full site media player only when saved
playback state or a user media action requires it.

Direct page loads, top-document media navigation, logout, back/forward history,
and continuous playback must remain correct.

### 2. CSS ownership split

Retain only universal layout, typography, navigation, forms, alerts, and shared
components in `main.css`. Move feature-exclusive rules to explicit stylesheets
owned by Void, WFL, Command Center, Shared Folder, and the media player. Each
template loads only its required feature sheets.

### 3. Stable asset fingerprint

Keep versioned paths, immutable one-year caching, and relative ES-module imports.
Replace commit-wide asset invalidation with a deterministic fingerprint derived
from the static-resource content set. A backend-only commit must preserve the
same asset URL; a changed asset set must produce a new URL.

### 4. Browser library consolidation

Create narrow shared modules for:

- WFL secondary navigation and stable address, cuisine, and rating formatting;
- alert/status show, hide, success, and error state;
- the existing URL-punctuation trimming implementation; and
- the established untrusted-text sanitization boundary.

Page-specific cards, state machines, network orchestration, and DOM ownership
remain in page modules. Shared helpers must use parameters or returned models,
not reach into page-global mutable state.

## Proposed `cbell-lib` Approach

### Immediate moves

- Move `StableCursor` and `StableCursorCodec` to
  `dev.christopherbell.libs.pagination`.
- Move generic Mongo lease documents, ownership errors, acquire/renew/release
  operations, renewable guards, and scheduled collector coordination to
  `dev.christopherbell.libs.mongo.lease`.
- Move `TestUtil` into a Gradle `testFixtures` source set and consume it through
  `testImplementation(testFixtures(project(":cbell-lib")))`.
- Move JJWT declarations into `website`, where `PermissionService` owns the
  production dependency.
- Because refreshed current source has only one production workflow-engine
  consumer, move the generic workflow package into the WFL feature. Retain it in
  `cbell-lib` only if pre-implementation inspection proves a second real
  production consumer.

### Conditional moves

Extract a bounded expiring-key store only after at least two rate-limit or
anti-abuse consumers use the same semantics. Extract a bounded outbound-body
reader only after at least two clients prove one transport-neutral contract.
Do not replace feature-specific response models with a generic page envelope
unless API schema names and documentation remain clear.

## Expected Files and Ownership

Likely backend hot-path files include:

- `website/src/main/java/dev/christopherbell/configuration/security/JwtAuthenticationFilter.java`;
- `website/src/main/java/dev/christopherbell/configuration/security/browser/BrowserSessionService.java`;
- `website/src/main/java/dev/christopherbell/message/conversation/ConversationService.java`;
- `website/src/main/java/dev/christopherbell/message/conversation/ConversationQueryRepository.java`;
- `website/src/main/java/dev/christopherbell/vehicle/nhtsa/decode/VehicleVinDecodeRateLimiter.java`;
- synchronous outbound client implementations under Canes, WFL, vehicle, post,
  and federation packages; and
- scheduled services and the existing configuration Mongo lease package.

Likely browser files include:

- `website/src/main/resources/static/js/app.js`;
- `website/src/main/resources/static/js/components/site-media-player.js`;
- WFL page scripts and new focused modules under `static/js/lib`;
- `website/src/main/resources/static/js/lib/util.js` and `feed-render.js`;
- `website/src/main/resources/static/css/main.css`; and
- feature templates and feature-owned stylesheets.

Likely library/build files include:

- `cbell-lib/build.gradle.kts`;
- `website/build.gradle.kts`;
- `cbell-lib/src/main/java/dev/christopherbell/libs/`;
- `cbell-lib/src/testFixtures/`; and
- consuming website imports and tests.

Exact edit ranges belong in the implementation plan after the written spec is
reviewed and current mainline is refreshed.

## Validation Plan

### Baseline

- Record the refreshed commit and clean isolated worktree.
- Measure the `app.js` transitive graph and CSS bytes by route.
- Record Mongo command/query counts for static requests, cookie authentication,
  conversation summaries, and selected feed/WFL endpoints.
- Record p50, p95, and p99 latency for representative anonymous and
  authenticated routes under a documented local workload.
- Record process-local store cardinality and scheduled-job ownership behavior.

### Focused automated verification

- Authentication filter tests with repository interactions asserted at zero for
  static assets.
- Browser-session revocation, rotation, legacy-session, concurrency, and
  write-coalescing tests.
- Query-count tests for conversation summaries and every issue-specific batching
  change.
- Bounded-cardinality, expiry, and concurrent-access tests for local stores.
- Oversized, slow, malformed, failed, and interrupted upstream response tests.
- Multi-instance lease contention, renewal, expiry, and ownership-loss tests.
- Node syntax and built-in JavaScript tests for conditional imports and shared
  UI helpers.
- Rendered-template tests proving route-specific CSS ownership.
- A deterministic asset-fingerprint verification proving backend-only changes
  preserve asset URLs and static changes rotate them.
- `:cbell-lib:test`, focused website tests, `:website:jsTest`, and full
  `:website:check`.

### Runtime and delivery verification

- Run the application with disposable data on a non-production port.
- Exercise anonymous pages, authenticated pages, static assets, conversations,
  VIN decoding, media playback, navigation history, and representative failure
  paths.
- Compare before-and-after measurements under the same workload.
- Deliver independent reviewable batches through pull request, required CI,
  merge, and production-safe verification.
- Do not change the live listener until alternate-port validation passes.

## Acceptance Criteria

1. Authenticated static asset requests perform zero session or account MongoDB
   operations.
2. Browser authentication uses one MongoDB round trip in its normal path, and
   interactive requests do not rewrite session state more often than the
   documented coalescing interval.
3. Account suspension, credential changes, role changes, deletion, logout, and
   explicit revocation invalidate sessions with tested semantics.
4. Conversation summary query count remains constant from one through fifty
   conversations.
5. Every changed local store and outbound body has a tested cardinality or byte
   bound and expiry or release behavior.
6. Every scheduled writer is classified, and every multi-instance-sensitive
   writer uses a tested durable claim or lease.
7. Simple pages parse at least 50 percent less unrelated JavaScript than the
   172,868-byte baseline graph. Measure the static transitive module graph
   loaded before user interaction on `/login`, `/signup`, `/vin-decoder`, and
   `/zip-coordinates`; each route must load no more than 86,434 raw bytes from
   the global graph unless a documented route requirement is approved.
8. Feature-exclusive CSS is absent from unrelated routes.
9. Backend-only releases reuse unchanged static asset URLs, while static changes
   rotate the asset fingerprint.
10. Stable cursor and Mongo lease primitives live in `cbell-lib`; shared test
    utilities use test fixtures; website-owned JWT dependencies live in
    `website`.
11. Browser WFL, alert, URL-trimming, and sanitization behavior has one tested
    shared implementation where semantics match.
12. Existing #1258-#1307 work is referenced rather than duplicated.
13. Focused tests, full `:website:check`, alternate-port acceptance, required
    pull-request checks, merge, and production-safe verification pass for every
    completed implementation batch.

## Risks and Mitigations

- **Authentication optimization weakens revocation:** centralize every
  security-state mutation, reject incomplete legacy sessions, and test races
  before removing the account lookup.
- **Lazy loading breaks continuous playback:** keep the routing shell minimal,
  test direct and in-shell navigation, and load full playback state only on a
  persisted session or media action.
- **CSS splitting changes visual precedence:** move one feature at a time and
  compare rendered pages at representative desktop and mobile widths.
- **Shared libraries become dumping grounds:** require two real consumers, a
  stable interface, and no feature-specific policy before each extraction.
- **Distributed limits add database load:** bound process-local stores first and
  introduce a distributed backend only when multi-instance deployment evidence
  justifies it.
- **Concurrent mainline campaign changes overlap:** refresh `origin/main`, reuse
  merged issue work, and re-plan edit ranges before implementation.
- **Security-remediation overlap:** preserve the validated security assessment's
  authority boundary and coordinate any outbound-client or authentication change
  with approved security remediation rather than weakening it incidentally.

## Delivery Sequence

1. Baseline measurements and regression-test seams.
2. Written-spec review, literal line-range implementation planning, and a
   separate user execution-authorization gate.
3. Static authentication bypass and browser-session operation reduction.
4. Conversation batching, VIN limiter bounds, and outbound-response bounds.
5. Conditional browser modules, feature CSS, and asset fingerprinting.
6. Behavior-preserving Java and browser library moves.
7. Remaining existing scaling issues in dependency-ordered campaign batches.
8. Full verification, pull-request review, merge, production acceptance, issue
   closure where applicable, and Builder closeout.

## Open Questions

None. The user approved the hot-path-first strategy, backend priorities,
frontend split, both library boundaries, and verification/rollout design on
2026-07-29. The written specification remains at the required user-review gate
before implementation planning.
