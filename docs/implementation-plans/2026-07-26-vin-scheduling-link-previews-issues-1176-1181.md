# VIN, Scheduling, and Link Previews Issues 1176-1181 Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use `superpowers:executing-plans` inline. Do not dispatch subagents. Execute every task test-first and preserve the dirty authoritative checkout.

**Goal:** Complete issues #1176-#1181 with versioned VIN freshness, ordered partial-success batch decode, production-safe RandomVIN scheduling, renewable distributed collector leases, and bounded SSRF-safe link previews.

**Architecture:** Keep VIN and link-preview ownership in their existing feature packages. Extend the shared Mongo lease boundary with a reusable renewable guard/coordinator and durable collector status, then use it from RandomVIN, NHTSA, Canes, and the existing WFL workflow. Persist versioned VIN and link-preview cache entries with explicit expiry; fetch previews through a manual-redirect transport that validates every destination before bounded reads.

**Tech Stack:** Java 25, Spring Boot 4.1, Spring Data MongoDB/MongoTemplate, Jakarta Validation, Java `HttpClient`, JSoup parsing, Gradle, JUnit 5, Mockito, AssertJ, and Mongo TTL indexes.

## Global Constraints

- Only comments authored by `azurras` may change issue scope or acceptance criteria; #1176-#1181 currently have no comments or attachments.
- Work only in `A:\Projects\christopherbell.dev-worktrees\vin-schedulers-link-preview-1176-1181`; do not modify or clean `A:\Projects\christopherbell.dev`.
- Use test-first RED/GREEN changes and repository-native Java/Spring/Mongo conventions.
- Test the packaged app on a non-8080 port and disposable Mongo database before production deployment.
- Keep existing `2026-05-09` single-VIN and vehicle-creation contracts compatible; add the partial-success batch decode as `2026-07-26`.
- Do not perform authenticated destructive production mutations during acceptance.

---

## Document Status

ready-for-execution

## Objective

Deliver the approved final campaign batch as one reviewed, tested, production-safe change set, close #1176-#1181, and leave the 58-issue campaign ready for final Builder closure.

## Inputs

- Approved campaign specification: `docs/specs/2026-07-25-complete-all-open-christopherbell-dev-issues.md`, Batch 7.
- GitHub issue bodies for #1176-#1181, all open with no comments, labels, or attachments as of 2026-07-26.
- Spoke base `abd2051e76155e5c01137ebec10c2d7550ec3556`; baseline `:website:check` passed in 2m31s.
- Existing `VehicleVinDecodeService`, NHTSA batch client/enrichment, RandomVIN guards, `MongoLeaseService`, WFL renewable lease behavior, Canes weekly collector, and JSoup preview client.

## Assumptions

- NHTSA vPIC remains the authoritative decoder and supports at most 50 VINs per upstream batch.
- MongoDB remains available for cache expiry, lease coordination, and durable run status.
- Link previews need metadata only from HTML/XHTML responses; other content types are not previewable.
- Production auto-deployment continues to validate a candidate release before listener rotation.

## Branch

`codex/vin-schedulers-link-preview-1176-1181`

Worktree: `A:\Projects\christopherbell.dev-worktrees\vin-schedulers-link-preview-1176-1181`

Base: `abd2051e76155e5c01137ebec10c2d7550ec3556`

## Goals

- Use VIN cache entries only when decoder version matches and `expiresOn` is future; stale entries refresh without extending freshness on failure.
- Return one ordered success/error result per submitted VIN while enforcing the configured batch maximum.
- Bind and validate RandomVIN enablement, initial delay, fixed delay, minimum safe delay, request timeouts, and lease duration; keep it disabled by default.
- Serialize RandomVIN import, NHTSA enrichment, Canes scheduled/manual collection, and WFL import through deterministic owner-scoped renewable leases with durable skipped-run evidence.
- Reject unsafe link-preview schemes, userinfo, localhost/private/link-local/multicast/reserved IPv4 and IPv6 addresses, DNS results, and redirect destinations.
- Bound preview redirects, connect/read/overall time, bytes, content types, metadata lengths, URLs per post, and success/failure cache lifetimes.

## Non-Goals

- Do not redesign the VIN decoder page or vehicle administration UI.
- Do not turn link previews into a general-purpose proxy, browser, image fetcher, or JavaScript renderer.
- Do not add Redis, ShedLock, Caffeine, or a new external dependency when MongoDB and Java platform APIs satisfy the contract.
- Do not lease unrelated short housekeeping schedulers such as post expiry or media playback polling.
- Do not change existing post rendering or stored `PostLinkPreview` response fields.

## Open Questions

None. The approved specification and current boundaries determine the implementation choices below.

## Code Changes

- Add explicit decoder version/expiry to VIN cache records and a token-weighted, ordered partial-success batch decoder.
- Validate typed VIN, RandomVIN, NHTSA, Canes, and link-preview configuration at startup.
- Add a shared renewable Mongo collector coordinator with durable lifecycle/skip status and adopt it in the scoped scheduled/manual collectors.
- Replace automatic JSoup network fetching with public-destination policy, manual redirects, bounded Java HTTP reads, and parse-only JSoup use.
- Persist bounded link-preview success/failure cache entries and add immutable V003 TTL/status indexes.
- Update feature READMEs, controller/security tests, focused suites, and packaged-runtime acceptance evidence.

## Files and Modules

- `vehicle/model/VehicleProperties.java`: typed, startup-validated VIN/scheduler/cache limits.
- `vehicle/nhtsa/decode/VehicleVinDecodeService.java`: single and batch freshness/decode orchestration.
- `configuration/mongo/lease/`: reusable renewable guard, collector coordinator, and durable status.
- `post/preview/`: public-address policy, bounded transport, cache, and metadata parser.
- `configuration/mongo/migration/V003EnsureVinPreviewCollectorIndexes.java`: immutable TTL/lookup indexes.
- Existing controllers expose only additive API behavior; existing service responses remain compatible.

## Task Breakdown

### Task 1: Version and expire VIN decode cache entries (#1176)

**Interfaces**

- Produces: `VehicleVinDecodeCache.isFresh(String, Instant)`, typed `vehicles.vin-decoder.decoder-version` and `cache-ttl`, and V003 VIN expiry index.
- Consumes: existing single-VIN `VehicleVinDecodeService.decode` and `VehicleVinDecodeCacheRepository`.

#### Code Edit 1.1
- File: `website/src/test/java/dev/christopherbell/vehicle/VehicleVinDecodeServiceTest.java`
- Lines: after 0
- Action: add

Proposed:
```java
@Test
void freshMatchingCacheAvoidsUpstreamButStaleOrOldVersionRefreshes() throws Exception {
  // Store matching/future, expired, and old-version entries against a fixed Clock.
  // Assert only the latter two call NHTSA and receive new refreshed/expires timestamps.
}

@Test
void failedStaleRefreshDoesNotRewriteOrReturnStaleData() {
  // Make the stale refresh fail and verify cacheRepository.save is never called.
}
```

Verification: run `\.\gradlew.bat :website:test --tests '*VehicleVinDecodeServiceTest'`; RED must show current cache entries ignore version/expiry.

#### Code Edit 1.2
- File: `website/src/main/java/dev/christopherbell/vehicle/model/VehicleVinDecodeCache.java`
- Lines: 1-37
- Action: add

Proposed:
```java
private String decoderVersion;
private Instant refreshedOn;
private Instant expiresOn;

public boolean isFresh(String expectedVersion, Instant now) {
  return expectedVersion.equals(decoderVersion) && expiresOn != null && expiresOn.isAfter(now);
}
```

Verification: model test partitions missing/mismatched version and equal/past/future expiry.

#### Code Edit 1.3
- File: `website/src/main/java/dev/christopherbell/vehicle/nhtsa/decode/VehicleVinDecodeService.java`
- Lines: 20-125
- Action: add

Proposed:
```java
private VehicleVinDecodeResponse cachedResponse(String vin) {
  var now = Instant.now(clock);
  return cacheRepository.findById(vin)
      .filter(entry -> entry.isFresh(decoderProperties.getDecoderVersion(), now))
      .map(VehicleVinDecodeCache::getResponse)
      .orElse(null);
}

private void saveCachedResponse(String vin, VehicleVinDecodeResponse response) {
  var now = Instant.now(clock);
  cacheRepository.save(VehicleVinDecodeCache.builder()
      .vin(vin)
      .decoderVersion(decoderProperties.getDecoderVersion())
      .refreshedOn(now)
      .expiresOn(now.plus(decoderProperties.getCacheTtl()))
      .response(response)
      .createdOn(now)
      .lastUpdatedOn(now)
      .build());
}
```

Verification: focused test turns GREEN and verifies a failed refresh preserves the stored stale entry unchanged.

#### Code Edit 1.4
- File: `website/src/main/java/dev/christopherbell/configuration/mongo/migration/V003EnsureVinPreviewCollectorIndexes.java`
- Lines: after 0
- Action: add

Proposed:
```java
@Component
public final class V003EnsureVinPreviewCollectorIndexes implements ApplicationMigration {
  public String id() { return "003-ensure-vin-preview-collector-indexes"; }
  public void apply(MongoTemplate mongo) {
    mongo.indexOps("vehicle_vin_decode_cache").createIndex(new Index()
        .on("expiresOn", Direction.ASC).expire(Duration.ZERO).named("vehicle_vin_cache_expiry"));
  }
}
```

Verification: `V003EnsureVinPreviewCollectorIndexesTest` verifies the named zero-second TTL index and immutable checksum.

### Task 2: Add ordered partial-success VIN batch decode (#1177)

**Interfaces**

- Produces: `VehicleVinDecodeBatchRequest`, `VehicleVinDecodeBatchEntry`, `VehicleVinDecodeBatchResponse`, and `POST /api/vehicles/2026-07-26/vin/decode/batch`.
- Consumes: Task 1 freshness rules, `NhtsaVinClient.decodeVins`, existing controller client-key resolution, and rate limiter.

#### Code Edit 2.1
- File: `website/src/test/java/dev/christopherbell/vehicle/VehicleVinDecodeServiceTest.java`
- Lines: after 0
- Action: add

Proposed:
```java
@Test
void batchPreservesInputOrderAcrossCachedDecodedInvalidAndMissingRows() throws Exception {
  var result = service.decodeBatch(new VehicleVinDecodeBatchRequest(List.of(
      VALID_CACHED, "bad", VALID_REMOTE, VALID_MISSING)), "ip:test");
  assertThat(result.results()).extracting(VehicleVinDecodeBatchEntry::submittedVin)
      .containsExactly(VALID_CACHED, "bad", VALID_REMOTE, VALID_MISSING);
  assertThat(result.results()).extracting(VehicleVinDecodeBatchEntry::status)
      .containsExactly("SUCCESS", "INVALID_VIN", "SUCCESS", "UPSTREAM_NO_RESULT");
}

@Test
void batchRejectsOnlyTheEnvelopeWhenConfiguredMaximumIsExceeded() {
  // maxBatchSize + 1 returns InvalidRequestException before rate limit or NHTSA.
}
```

Verification: focused RED demonstrates no batch service exists.

#### Code Edit 2.2
- File: `website/src/main/java/dev/christopherbell/vehicle/model/VehicleVinDecodeBatchRequest.java`
- Lines: after 0
- Action: add

Proposed:
```java
public record VehicleVinDecodeBatchRequest(@NotNull List<String> vins) {}
```

Verification: request-model test preserves submitted order and rejects a null list before service entry.

#### Code Edit 2.3
- File: `website/src/main/java/dev/christopherbell/vehicle/model/VehicleVinDecodeBatchEntry.java`
- Lines: after 0
- Action: add

Proposed:
```java
public record VehicleVinDecodeBatchEntry(
    int index,
    String submittedVin,
    String normalizedVin,
    String status,
    VehicleVinDecodeResponse decoded,
    String errorCode,
    String errorMessage) {}
```

Verification: JSON/model test proves success and error entries expose mutually exclusive decoded/error fields with stable indexes.

#### Code Edit 2.4
- File: `website/src/main/java/dev/christopherbell/vehicle/model/VehicleVinDecodeBatchResponse.java`
- Lines: after 0
- Action: add

Proposed:
```java
public record VehicleVinDecodeBatchResponse(
    int submittedCount,
    int successCount,
    int errorCount,
    List<VehicleVinDecodeBatchEntry> results) {}
```

Verification: response-model test verifies submitted/success/error counts agree with the ordered immutable result list.

#### Code Edit 2.5
- File: `website/src/main/java/dev/christopherbell/vehicle/nhtsa/decode/VehicleVinDecodeService.java`
- Lines: 48-170
- Action: add

Proposed:
```java
public VehicleVinDecodeBatchResponse decodeBatch(
    VehicleVinDecodeBatchRequest request, String clientKey) throws InvalidRequestException {
  validateBatchEnvelope(request);
  rateLimiter.check(rateLimitKey(clientKey), request.vins().size());
  // Preserve every submitted position. Validate individually, use fresh cache entries,
  // send unique stale/missing valid VINs through one bounded NHTSA batch, then map each
  // row to SUCCESS, INVALID_VIN, UPSTREAM_NO_RESULT, or UPSTREAM_UNAVAILABLE.
}
```

Verification: tests assert order, duplicates, cache reuse, invalid partitions, partial NHTSA rows, one batch call, safe upstream errors, and configured max.

#### Code Edit 2.6
- File: `website/src/main/java/dev/christopherbell/vehicle/nhtsa/decode/VehicleVinDecodeRateLimiter.java`
- Lines: 14-34
- Action: add

Proposed:
```java
public void check(String key, long tokens) {
  if (tokens < 1 || !bucket(key).tryConsume(tokens)) {
    throw new VehicleVinDecodeRateLimitException("Too many VIN decode requests. Please try again later.");
  }
}
```

Verification: limiter test proves a batch consumes its VIN count, not one request token.

#### Code Edit 2.7
- File: `website/src/main/java/dev/christopherbell/vehicle/VehicleController.java`
- Lines: 98-134
- Action: add

Proposed:
```java
@PostMapping(value = V20260726 + "/vin/decode/batch", consumes = APPLICATION_JSON_VALUE)
public ResponseEntity<Response<VehicleVinDecodeBatchResponse>> decodeVinBatch(
    @RequestBody VehicleVinDecodeBatchRequest request, HttpServletRequest servletRequest) {
  var clientKey = clientKey(servletRequest, clientIpResolver.resolveClientIp(servletRequest));
  return ResponseEntity.ok(Response.<VehicleVinDecodeBatchResponse>builder()
      .payload(vehicleVinDecodeService.decodeBatch(request, clientKey)).success(true).build());
}
```

Verification: controller tests prove anonymous/authenticated client keys, ordered 200 partial results, malformed JSON 400, oversized envelope 400, and 429 propagation.

#### Code Edit 2.8
- File: `website/src/main/java/dev/christopherbell/configuration/security/SecurityConfig.java`
- Lines: 80-100
- Action: add

Proposed:
```java
"POST:/api/vehicles" + APIVersion.V20260726 + "/vin/decode/batch",
```

Verification: `SecurityConfigTest` proves only the exact additive POST is public; adjacent vehicle administration routes remain protected.

### Task 3: Validate production-safe RandomVIN and VIN scheduler configuration (#1178)

**Interfaces**

- Produces: typed `initialDelay`, `fixedDelay`, `minimumSafeDelay`, `leaseDuration`, cache/batch limits, and startup validation.
- Consumes: existing `VehicleProperties`, application profiles, RandomVIN client timeouts, and Task 4 lease coordinator.

#### Code Edit 3.1
- File: `website/src/test/java/dev/christopherbell/vehicle/VehiclePropertiesTest.java`
- Lines: after 0
- Action: add

Proposed:
```java
@Test
void rejectsEnabledRandomVinBelowMinimumDelayOrLeaseBelowRequestBudget() {
  // Validate fixedDelay >= minimumSafeDelay and leaseDuration > requestTimeout.
}

@Test
void checkedInDefaultsAreDisabledAndProductionSafe() {
  // Bind application.yml and assert disabled, PT1M initial, PT10M fixed, PT1M minimum.
}
```

Verification: RED shows current unvalidated 1000ms fixed delay.

#### Code Edit 3.2
- File: `website/src/main/java/dev/christopherbell/vehicle/model/VehicleProperties.java`
- Lines: 1-58
- Action: add

Proposed:
```java
@ConfigurationProperties(prefix = "vehicles")
@Validated
public class VehicleProperties {
  @Valid private NhtsaVin nhtsaVin = new NhtsaVin();
  @Valid private RandomVin randomVin = new RandomVin();
  @Valid private VinDecoder vinDecoder = new VinDecoder();

  @AssertTrue(message = "enabled RandomVIN fixed delay must meet minimum-safe-delay")
  public boolean isRandomVinScheduleSafe() { /* disabled or fixed >= minimum */ }
}
```

Verification: validation tests cover positive durations, batch 1..50, nonblank decoder version, enabled safety relation, and lease/request relation.

#### Code Edit 3.3
- File: `website/src/main/resources/application.yml`
- Lines: 574-603
- Action: add

Proposed:
```yaml
vehicles:
  nhtsa-vin:
    initial-delay: 1m
    fixed-delay: 1h
    lease-duration: 2m
  random-vin:
    enabled: false
    initial-delay: 1m
    fixed-delay: 10m
    minimum-safe-delay: 1m
    lease-duration: 2m
    max-calls-per-day: 144
  vin-decoder:
    decoder-version: nhtsa-vpic-2026-07-26
    cache-ttl: 30d
    max-batch-size: 20
```

Verification: configuration test binds exact defaults; profile tests keep scheduled mutations disabled in `test` and `deploy-smoke`.

#### Code Edit 3.4
- File: `website/src/main/java/dev/christopherbell/vehicle/randomvin/importing/RandomVinImportService.java`
- Lines: 75-85
- Action: add

Proposed:
```java
@Scheduled(
    initialDelayString = "${vehicles.random-vin.initial-delay}",
    fixedDelayString = "${vehicles.random-vin.fixed-delay}")
public void importRandomVin() { /* existing enablement and guards */ }
```

Verification: scheduler metadata/configuration test proves there is no 1-second production default.

### Task 4: Share renewable Mongo leases and durable skip status across collectors (#1179)

**Interfaces**

- Produces: `RenewingMongoLease.checkpoint()`, `ScheduledCollectorCoordinator.run(...)`, `ScheduledCollectorRun`, and deterministic names.
- Consumes: `MongoLeaseService.tryAcquire/renew/release`, `Clock`, Task 3 lease durations, existing WFL lease status, and domain workflows.

#### Code Edit 4.1
- File: `website/src/test/java/dev/christopherbell/configuration/mongo/lease/ScheduledCollectorCoordinatorTest.java`
- Lines: after 0
- Action: add

Proposed:
```java
@Test
void contentionPersistsSkippedLockedWithoutInvokingCollector() { }

@Test
void longRunRenewsAtHalfDurationAndReleasesExactOwner() { }

@Test
void failureStoresOnlyAllowlistedCategoryAndStillReleases() { }
```

Verification: RED shows the coordinator types are absent.

#### Code Edit 4.2
- File: `website/src/main/java/dev/christopherbell/configuration/mongo/lease/RenewingMongoLease.java`
- Lines: after 0
- Action: add

Proposed:
```java
public final class RenewingMongoLease {
  public void checkpoint() {
    var now = Instant.now(clock);
    if (now.isBefore(renewOn)) return;
    if (!leases.renew(name, ownerToken, now, now.plus(duration))) {
      throw new LeaseLostException("Scheduled collector lease was lost.");
    }
    renewOn = now.plus(duration.dividedBy(2));
  }
}
```

Verification: guard test proves no early renewal, half-duration renewal, lost-owner failure, and renewed deadline movement.

#### Code Edit 4.3
- File: `website/src/main/java/dev/christopherbell/configuration/mongo/lease/ScheduledCollectorCoordinator.java`
- Lines: after 0
- Action: add

Proposed:
```java
public <T> ScheduledCollectorResult<T> run(
    String name, Duration duration, String trigger,
    Function<RenewingMongoLease, T> collector) {
  // UUID owner; atomic acquire; RUNNING/SKIPPED_LOCKED/SUCCEEDED/FAILED state;
  // safe error category; exact-owner release in finally.
}
```

Verification: coordinator test verifies acquire, durable RUNNING/terminal state, safe exception translation, and release in `finally`.

#### Code Edit 4.4
- File: `website/src/main/java/dev/christopherbell/configuration/mongo/lease/ScheduledCollectorRun.java`
- Lines: after 0
- Action: add

Proposed:
```java
@Document("scheduled_collector_runs")
public class ScheduledCollectorRun {
  @Id private String name;
  private String status;
  private String trigger;
  private Instant startedOn;
  private Instant completedOn;
  private Instant skippedOn;
  private String safeErrorCategory;
}
```

Verification: focused coordinator tests prove owner scoping, renewal, contention, safe status, and release.

#### Code Edit 4.5
- File: `website/src/main/java/dev/christopherbell/vehicle/randomvin/importing/RandomVinImportService.java`
- Lines: 29-135
- Action: add

Proposed:
```java
private static final String COLLECTOR = "vehicle-random-vin-import";

public void importRandomVin() {
  if (!properties.isEnabled()) return;
  coordinator.run(COLLECTOR, properties.getLeaseDuration(), "SCHEDULED", lease -> {
    lease.checkpoint();
    return importOneVin();
  });
}
```

Verification: RandomVIN tests assert contention makes no robots/client/repository call and records `SKIPPED_LOCKED`.

#### Code Edit 4.6
- File: `website/src/main/java/dev/christopherbell/vehicle/nhtsa/enrichment/NhtsaVinEnrichmentService.java`
- Lines: 25-140
- Action: add

Proposed:
```java
private static final String COLLECTOR = "vehicle-nhtsa-enrichment";

public void enrichStoredVins() {
  if (!properties.isEnabled()) return;
  coordinator.run(COLLECTOR, properties.getLeaseDuration(), "SCHEDULED", lease -> {
    for (var batch : batches(dueVehicles())) {
      lease.checkpoint();
      enrichVehicleBatch(state, batch);
    }
    lease.checkpoint();
    return null;
  });
}
```

Verification: NHTSA tests assert one deterministic lease, renewal before later batches, lost-lease abort, and exact-owner release.

#### Code Edit 4.7
- File: `website/src/main/java/dev/christopherbell/canesboxtracker/CanesBoxTrackerService.java`
- Lines: 25-110
- Action: add

Proposed:
```java
private static final String COLLECTOR = "canes-box-weekly-collection";

private ScheduledCollectorResult<CanesBoxPriceSnapshot> collectWithLease(String trigger) {
  return coordinator.run(COLLECTOR, properties.getCollection().getLeaseDuration(), trigger,
      lease -> collectWeek(currentWeekStart(), lease));
}
```

Verification: scheduled contention records `SKIPPED_LOCKED`; admin contention returns a safe 409/503 instead of overlapping; each metro fetch checkpoints lease ownership.

#### Code Edit 4.8
- File: `website/src/main/java/dev/christopherbell/whatsforlunch/restaurant/importing/RestaurantImportLeaseGuard.java`
- Lines: 1-61
- Action: add

Proposed:
```java
// Replace feature-specific renewal math with delegation to RenewingMongoLease;
// retain the Restaurant workflow's existing API and status semantics.
```

Verification: existing long-apply WFL tests remain GREEN and prove the shared abstraction did not weaken Batch 6.

#### Code Edit 4.9
- File: `website/src/main/java/dev/christopherbell/canesboxtracker/model/CanesBoxTrackerProperties.java`
- Lines: 1-58
- Action: add

Proposed:
```java
@Data
public static class CollectionSchedule {
  private String zone = "America/Chicago";
  private Duration leaseDuration = Duration.ofMinutes(2);
}
```

Verification: properties test requires a positive lease duration longer than one configured request timeout.

#### Code Edit 4.10
- File: `website/src/main/resources/application.yml`
- Lines: 109-125
- Action: add

Proposed:
```yaml
canes-box-tracker:
  collection:
    cron: "0 0 6 * * MON"
    zone: America/Chicago
    lease-duration: 2m
```

Verification: checked configuration binds the same schedule plus a safe renewable lease duration.

### Task 5: Enforce public destinations and bounded manual redirects (#1180)

**Interfaces**

- Produces: `PostLinkPreviewDestinationPolicy.requirePublic(URI)`, injectable `LinkPreviewDnsResolver`, and `BoundedLinkPreviewHttpClient.fetch(URI)`.
- Consumes: typed preview properties and Java `HttpClient` configured with redirects disabled.

#### Code Edit 5.1
- File: `website/src/test/java/dev/christopherbell/post/PostLinkPreviewDestinationPolicyTest.java`
- Lines: after 0
- Action: add

Proposed:
```java
@ParameterizedTest
@ValueSource(strings = {
  "file:///etc/passwd", "http://user@example.com", "http://127.0.0.1",
  "http://169.254.169.254", "http://10.0.0.1", "http://[::1]",
  "http://[fc00::1]", "http://[fe80::1]", "http://[2001:db8::1]"
})
void blocksUnsafeLiteralDestinations(String url) { }

@Test
void blocksHostnameWhenAnyDnsAnswerIsPrivate() { }
```

Verification: RED shows current URI check does not cover userinfo/reserved/documentation networks or injected DNS.

#### Code Edit 5.2
- File: `website/src/main/java/dev/christopherbell/post/preview/PostLinkPreviewDestinationPolicy.java`
- Lines: after 0
- Action: add

Proposed:
```java
public URI requirePublic(URI uri) {
  // Require absolute http/https, no userinfo, nonblank host, valid port.
  // Resolve every address and reject IPv4/IPv6 unspecified, loopback, private,
  // shared, link-local, documentation, benchmark, multicast, and reserved ranges.
  return uri.normalize();
}
```

Verification: destination-policy test covers unsafe schemes/userinfo, all blocked literal/DNS ranges, mixed answers, and public IPv4/IPv6.

#### Code Edit 5.3
- File: `website/src/test/java/dev/christopherbell/post/BoundedLinkPreviewHttpClientTest.java`
- Lines: after 0
- Action: add

Proposed:
```java
@Test
void revalidatesEveryRedirectBeforeSecondRequest() { }

@Test
void rejectsRedirectLoopMissingLocationAndRedirectLimit() { }

@Test
void rejectsOversizeUnsupportedContentAndOverallTimeout() { }
```

Verification: focused transport RED names each missing redirect/size/content/timeout guard before implementation.

#### Code Edit 5.4
- File: `website/src/main/java/dev/christopherbell/post/preview/BoundedLinkPreviewHttpClient.java`
- Lines: after 0
- Action: add

Proposed:
```java
public LinkPreviewHttpResponse fetch(URI initial) {
  var current = policy.requirePublic(initial);
  for (int redirects = 0; redirects <= properties.getMaxRedirects(); redirects++) {
    var response = transport.send(current, remainingOverallBudget());
    if (isRedirect(response.status())) {
      current = policy.requirePublic(current.resolve(requireLocation(response)));
      continue;
    }
    return requireHtmlAndReadAtMost(response, properties.getMaxResponseBytes());
  }
  throw new LinkPreviewFetchException("REDIRECT_LIMIT");
}
```

Verification: tests prove policy validation happens before the first request and every redirect; no request is made to rejected targets.

#### Code Edit 5.5
- File: `website/src/main/java/dev/christopherbell/post/preview/JsoupPostLinkPreviewClient.java`
- Lines: 1-130
- Action: add

Proposed:
```java
public Optional<PostLinkPreview> fetch(String url) {
  var response = boundedHttp.fetch(URI.create(url));
  var document = Jsoup.parse(response.body(), response.finalUri().toString());
  return toPreview(response.finalUri(), document);
}
```

Verification: parser tests retain relative image resolution and apply configured title/description/image URL length caps.

### Task 6: Cache bounded preview successes and failures (#1181)

**Interfaces**

- Produces: `PostLinkPreviewProperties`, `PostLinkPreviewCacheEntry`, repository/store, TTL index, and cache-aware `PostLinkPreviewService`.
- Consumes: Task 5 bounded client and existing post creation/editing services.

#### Code Edit 6.1
- File: `website/src/test/java/dev/christopherbell/post/PostLinkPreviewServiceTest.java`
- Lines: after 0
- Action: add

Proposed:
```java
@Test
void recentFailureSkipsOutboundWorkUntilFailureExpiry() { }

@Test
void successCacheReturnsPreviewUntilSuccessExpiryThenRefreshes() { }

@Test
void resolvesOnlyConfiguredMaximumDistinctUrlsInFirstSeenOrder() { }
```

Verification: RED shows current service always repeats failed outbound fetches.

#### Code Edit 6.2
- File: `website/src/main/java/dev/christopherbell/post/preview/PostLinkPreviewProperties.java`
- Lines: after 0
- Action: add

Proposed:
```java
@ConfigurationProperties(prefix = "posts.link-previews")
@Validated
public class PostLinkPreviewProperties {
  @Min(0) @Max(5) private int maxRedirects = 3;
  @Min(1024) @Max(1048576) private int maxResponseBytes = 262144;
  private Duration connectTimeout = Duration.ofSeconds(2);
  private Duration requestTimeout = Duration.ofSeconds(3);
  private Duration overallTimeout = Duration.ofSeconds(5);
  private Duration successTtl = Duration.ofDays(7);
  private Duration failureTtl = Duration.ofMinutes(15);
  @Min(1) @Max(5) private int maxUrlsPerPost = 3;
  @Min(1) @Max(500) private int maxTitleLength = 200;
  @Min(1) @Max(2000) private int maxDescriptionLength = 500;
  @Min(1) @Max(4096) private int maxImageUrlLength = 2048;
}
```

Verification: properties test binds checked defaults and rejects invalid timeout, redirect, byte, URL-count, metadata, and TTL bounds.

#### Code Edit 6.3
- File: `website/src/main/java/dev/christopherbell/post/preview/PostLinkPreviewCacheEntry.java`
- Lines: after 0
- Action: add

Proposed:
```java
@Document("post_link_preview_cache")
public class PostLinkPreviewCacheEntry {
  @Id private String normalizedUrl;
  private String outcome;
  private PostLinkPreview preview;
  private String safeFailureCategory;
  private Instant completedOn;
  private Instant expiresOn;
}
```

Verification: cache-entry test proves only SUCCESS may contain preview data and only FAILURE may contain a safe failure category.

#### Code Edit 6.4
- File: `website/src/main/java/dev/christopherbell/post/preview/PostLinkPreviewService.java`
- Lines: 1-65
- Action: add

Proposed:
```java
private Optional<PostLinkPreview> resolve(String normalizedUrl) {
  var cached = cache.findFresh(normalizedUrl, Instant.now(clock));
  if (cached.isPresent()) return cached.get().previewResult();
  try {
    var preview = client.fetch(normalizedUrl);
    cache.saveSuccessOrFailure(normalizedUrl, preview, now, properties);
    return preview;
  } catch (LinkPreviewFetchException failure) {
    cache.saveFailure(normalizedUrl, failure.safeCategory(), now.plus(properties.getFailureTtl()));
    return Optional.empty();
  }
}
```

Verification: tests verify fresh success/failure cache paths make zero outbound calls, expiry refreshes, cache write failure degrades safely, and only safe categories persist.

#### Code Edit 6.5
- File: `website/src/main/java/dev/christopherbell/configuration/mongo/migration/V003EnsureVinPreviewCollectorIndexes.java`
- Lines: after 0
- Action: add

Proposed:
```java
mongo.indexOps("post_link_preview_cache").createIndex(new Index()
    .on("expiresOn", Direction.ASC).expire(Duration.ZERO).named("post_link_preview_cache_expiry"));
mongo.indexOps("scheduled_collector_runs").createIndex(new Index()
    .on("status", Direction.ASC).on("completedOn", Direction.DESC)
    .named("scheduled_collector_status_completed"));
```

Verification: V003 test asserts all three named indexes and migration checksum; migration runner test applies V003 once.

#### Code Edit 6.6
- File: `website/src/main/resources/application.yml`
- Lines: 103-138
- Action: add

Proposed:
```yaml
posts:
  link-previews:
    connect-timeout: 2s
    request-timeout: 3s
    overall-timeout: 5s
    max-redirects: 3
    max-response-bytes: 262144
    allowed-content-types: [text/html, application/xhtml+xml]
    max-urls-per-post: 3
    max-title-length: 200
    max-description-length: 500
    max-image-url-length: 2048
    success-ttl: 7d
    failure-ttl: 15m
```

Verification: properties test partitions invalid timeout relationships, sizes, redirects, lengths, TTLs, and content types.

### Task 7: Documentation, regression gate, runtime acceptance, and publication

**Interfaces**

- Consumes: Tasks 1-6.
- Produces: final READMEs, verified branch, test report, reviewed PR, production acceptance, issue closure, and campaign closeout evidence.

#### Code Edit 7.1
- File: `website/src/main/java/dev/christopherbell/vehicle/vin/README.md`
- Lines: 1-12
- Action: add

Proposed:
```markdown
Document decoder version/expiry semantics, ordered partial-success batch results,
configured maximum, rate-cost behavior, and stale-refresh failure handling.
```

Verification: VIN README contains every exact configuration key and the `2026-07-26` endpoint path.

#### Code Edit 7.2
- File: `website/src/main/java/dev/christopherbell/vehicle/randomvin/README.md`
- Lines: 1-14
- Action: add

Proposed:
```markdown
Document disabled-by-default behavior, PT1M initial delay, PT10M fixed delay,
minimum-safe-delay validation, daily cap, robots policy, and distributed lease status.
```

Verification: RandomVIN README states disabled-by-default and exact safe checked durations.

#### Code Edit 7.3
- File: `website/src/main/java/dev/christopherbell/post/README.md`
- Lines: 1-91
- Action: add

Proposed:
```markdown
Document public-address validation, manual redirect revalidation, bounded HTML reads,
metadata caps, and success/failure TTL cache behavior.
```

Verification: documentation contains exact property names and makes no promise of JavaScript rendering or image proxying.

#### Code Edit 7.4
- File: `website/src/test/java/dev/christopherbell/configuration/SecurityConfigTest.java`
- Lines: after 0
- Action: add

Proposed:
```java
// Assert the additive batch VIN decoder route has the intended anonymous POST boundary;
// collector status remains protected/internal and no preview proxy route is added.
```

Verification: full controller/security tests pass.

## Risks

- DNS can change between policy resolution and the HTTP client's connection. Mitigation: resolve and reject every returned address immediately before each request, disable automatic redirects, revalidate every redirect target, use short deadlines, and never expose fetched bytes as a proxy response.
- Mongo TTL deletion is asynchronous. Mitigation: application reads enforce `expiresOn` directly; TTL indexes are cleanup only.
- Batch decode could amplify anonymous outbound work. Mitigation: configured max 20, token-weighted per-client rate limiting, fresh-cache reuse, one upstream batch for unique misses, and existing cooldown.
- A collector can run longer than its initial lease. Mitigation: reusable half-duration renewal checkpoints before each NHTSA batch/Canes metro/WFL mutation and final ownership verification.
- Link-preview failure caching can hide recovery briefly. Mitigation: short 15-minute failure TTL, safe category only, and no extension on cache reads.
- Tight RandomVIN validation could break startup. Mitigation: checked defaults are disabled/safe and profile binding tests run before publication.

## Validation

- Witness focused RED before production edits for cache freshness, batch partial results, configuration safety, lease contention/renewal, destination policy, redirects/limits, and failure caching.
- Run each owning Java test class after its GREEN change, plus existing WFL lease regressions after extracting the shared guard.
- Run JavaScript tests even though no presentation redesign is planned, then run the complete `:website:check` and `git diff --check`.
- Exercise a packaged candidate on port 8092 with an exact disposable Mongo database, inspect new indexes/state, stop only the candidate PID, and drop only that database.
- Require independent diff review, every PR gate, squash merge, main-branch gates, native listener rotation, live public/protected smoke checks, and issue closure.

## Unit Testing

- `VehicleVinDecodeServiceTest`: fresh/stale/version mismatch, failed refresh, ordered batch results, duplicate positions, cache mix, missing rows, upstream failure, and max batch.
- `VehiclePropertiesTest`: checked defaults, duration/limit partitions, enabled RandomVIN safety relation, and lease/request budget.
- `ScheduledCollectorCoordinatorTest`: contention, UUID owner, renewal, lost lease, exact release, status lifecycle, and safe errors.
- Existing RandomVIN/NHTSA/Canes/WFL tests: deterministic lock names, skip behavior, renewal checkpoints, and manual/scheduled equivalence.
- `PostLinkPreviewDestinationPolicyTest`: schemes, userinfo, literal/DNS IPv4 and IPv6 blocked ranges, mixed DNS answers, and public allow cases.
- `BoundedLinkPreviewHttpClientTest`: manual redirects, per-hop validation, missing/looping redirects, timeout budgets, content types, content length, streamed overrun, and bounded body.
- `PostLinkPreviewServiceTest`: URL limit/order, success/failure cache TTL, zero-repeat failure, safe degradation, and metadata lengths.
- V003 migration test: VIN/link-preview TTL and collector status indexes.

## Local Testing

1. Capture focused RED for every task before production code.
2. Run focused Java classes after each slice and keep WFL lease regressions green.
3. Run `:website:jsTest`, `:website:check`, and `git diff --check`; baseline at `abd2051e` passed in 2m31s with 233 JavaScript tests.
4. Build the executable JAR and run on port 8092 with a unique disposable Mongo database and `local,deploy-smoke` profiles.
5. Exercise `/`, `/tools`, `/vin-decoder`, single decode validation, additive batch decode with mixed invalid/valid inputs, oversize batch rejection, and protected collection state.
6. Inspect disposable Mongo records/indexes for decoder version/expiry, preview failure TTL, collector status, and released leases without contacting private targets.
7. Stop only the candidate PID, drop only the exact disposable database, and prove production 8080 stayed healthy.
8. Require independent diff review, all GitHub platform/Dependency Review/CodeQL gates, squash merge, native listener rotation, public/protected production smoke, and closure of #1176-#1181.

## Rollback or Recovery

- Before merge, revert only final-batch commits in the isolated worktree.
- Cache/schema additions are backward compatible; older application code ignores new fields/collections.
- Lost/expired collector leases fail closed and a later scheduled run can retry after release/expiry.
- Link-preview failures return no preview; post creation/editing remains available.
- After merge, native deployment retains the previous release and rolls back if candidate or production smoke fails.

## Completion Criteria

- Each issue #1176-#1181 has direct automated and packaged-runtime evidence matching the approved specification.
- Full local checks, independent review, Ubuntu/macOS/Windows, Dependency Review, underlying CodeQL, and aggregate CodeQL pass at final head.
- PR is squash-merged, production rotates to the merge, public pages/VIN routes remain healthy, protected boundaries deny anonymous callers, and all six issues close.
- Builder contains the final test report, spoke update/review, session memory, completed plan/spec/ledger, work closure, refreshed indexes, and no remaining open campaign issues.
