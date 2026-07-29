# Website-Wide Security Remediation Plan

## Document Status

ready-for-execution

## Objective

Fix all 17 reportable security findings from the repository-wide and freshness-delta reviews of `azurras/christopherbell.dev`, include the approved affirmative federation-consent hardening, prove each changed boundary test-first, and deliver the result through PR, CI, merge, production-safe verification, issue closure, and Builder closeout.

## Goals

- Make authentication and authorization reflect current account security state.
- Upgrade password verification without locking out legacy accounts.
- Remove public error, proxy, WFL, URL, SSRF, and browser-state boundary failures.
- Enforce federation post eligibility, account approval, and affirmative signup enrollment.
- Pin all executable build inputs.
- Preserve the authoritative dirty checkout and validate the packaged application away from port 8080 before production changes.

## Inputs

- Spec: `docs/specs/2026-07-29-website-wide-security-remediation.md`.
- Repository-wide scan at `f77c5f5bb644cc75cf98b27e722efdc00cd036f1`: 15 reportable findings.
- Complete 93-file ActivityPub delta scan through `6c1501070ff518bc040583c4576c2df201dcd3ed`: two reportable low-severity findings and one approved self-only privacy hardening item.
- User decision: approved; start implementation without another design checkpoint.
- Official current guidance: PBKDF2-HMAC-SHA256 at 600,000 iterations, strict Gradle dependency verification, Gradle distribution checksum pinning, and full-SHA GitHub Action pinning.

## Branch

Use `codex/security-audit-20260728` from refreshed `origin/main` in `A:\Projects\christopherbell.dev-worktrees\security-audit-20260728`. Do not modify `A:\Projects\christopherbell.dev`.

## Non-Goals

- Do not change the three operator/admin-only candidates rejected by the repository-wide scan.
- Do not redesign ActivityPub transport, signing, encrypted-key storage, or retry policy; those controls passed review.
- Do not weaken CSRF, proxy trust, filesystem ACLs, or production listener isolation.
- Do not add unrelated features or broad cleanup.

## Assumptions

- Mongo account IDs remain indexed and available to bearer authentication.
- The local reverse proxy reaches the application from loopback.
- Existing Reactor Netty support can provide hostname-preserving, approved-IP-pinned HTTP transport.
- Existing focused Java and JavaScript test harnesses remain authoritative for RED/GREEN evidence.

## Open Questions

None. Any source drift or implementation discovery that invalidates these boundaries requires a plan update before continuing that task.

## Task Breakdown

### Task 1 - Current account authorization and versioned password verifiers

Sequence / dependencies:

- Runs first because later account moderation and permission tests depend on one current security-state invariant.

Expected files or modules:

- `website/src/main/java/dev/christopherbell/account/**`
- `website/src/main/java/dev/christopherbell/configuration/security/JwtAuthenticationFilter.java`
- `website/src/main/java/dev/christopherbell/permission/PermissionService.java`
- `cbell-lib/src/main/java/dev/christopherbell/libs/security/PasswordUtil.java`
- focused account, JWT, password-reset, moderation, and browser-session tests

Implementation notes:

- Required skill: `write-jane-street-style-code` before any code edits; execution must invoke it and apply its Java, testing, invariant, and reviewability rules.
- Before-Edit Brief:
  - Behavior: bearer requests reload the account and reject stale, inactive, unapproved, or deleted state; all public login failures are indistinguishable; successful legacy-password login upgrades the verifier.
  - Invariants: current role and permissions come only from persistence; security-sensitive mutations invalidate existing bearer tokens; legacy accounts remain usable only after a correct password.
  - Boundary/API: preserve the existing login/token HTTP shapes except for uniform failure responses; add versioned persistence fields compatibly with missing legacy values.
  - Effects and failures: one indexed account lookup per bearer request; password rehash saves only after successful verification; missing/stale versions fail closed.
  - Tests and evidence: first add failing JWT revocation, login-equivalence, constant-time/version format, and transparent-upgrade tests; capture RED before production edits and GREEN afterward.
- Add a monotonic account security version to the account/JWT contract and advance it for password, role, status, approval, shared-folder permission, and music permission changes.
- Use a fixed dummy verifier for unknown-email login so the public path performs equivalent PBKDF2 work.

#### Code Edit 1.1

- File: `website/src/main/java/dev/christopherbell/configuration/security/JwtAuthenticationFilter.java`
- Lines: 85-94
- Action: replace

Current:

```java
    try {
      if (bearerToken != null && Objects.nonNull(PermissionService.validateToken(bearerToken))) {
        Authentication authenticationToken = getAuthentication(bearerToken);
        SecurityContextHolder.getContext().setAuthentication(authenticationToken);
        if (authenticationToken.isAuthenticated()) {
          chain.doFilter(request, response);
        } else {
          response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
        }
        return;
      }
```

Proposed:

```java
    try {
      if (bearerToken != null) {
        var claims = PermissionService.validateToken(bearerToken);
        var account = accountRepository.findById(claims.getSubject())
            .filter(AccountSecurityState::mayAuthenticate)
            .filter(current -> AccountSecurityState.matchesTokenVersion(current, claims))
            .orElseThrow(InvalidTokenException::new);
        Authentication authenticationToken = getAuthentication(account);
        SecurityContextHolder.getContext().setAuthentication(authenticationToken);
        chain.doFilter(request, response);
        return;
      }
```

Verification:

- `./gradlew.bat :website:test --tests '*JwtAuthenticationFilterTest' --tests '*AccountAuthenticationServiceTest' --tests '*PasswordResetServiceTest' --no-daemon`

#### Code Edit 1.2

- File: `cbell-lib/src/main/java/dev/christopherbell/libs/security/PasswordUtil.java`
- Lines: 16-48
- Action: replace

Current:

```java
  private static final int SALT_LENGTH = 16;
  private static final int HASH_ITERATIONS = 65536;
  private static final int HASH_KEY_LENGTH = 256;

  public static String generateSalt() {
    SecureRandom secureRandom = new SecureRandom();
    byte[] salt = new byte[SALT_LENGTH];
    secureRandom.nextBytes(salt);
    return Base64.getEncoder().encodeToString(salt);
  }

  public static String hashPassword(String password, String salt)
      throws NoSuchAlgorithmException, InvalidKeySpecException {
    PBEKeySpec spec = new PBEKeySpec(
        password.toCharArray(), Base64.getDecoder().decode(salt), HASH_ITERATIONS, HASH_KEY_LENGTH);
    SecretKeyFactory factory = SecretKeyFactory.getInstance("PBKDF2WithHmacSHA256");
    byte[] hash = factory.generateSecret(spec).getEncoded();
    return Base64.getEncoder().encodeToString(hash);
  }

  public static boolean verifyPassword(String password, String salt, String storedHash)
      throws NoSuchAlgorithmException, InvalidKeySpecException {
    String computedHash = hashPassword(password, salt);
    return computedHash.equals(storedHash);
  }
```

Proposed:

```java
  private static final int CURRENT_ITERATIONS = 600_000;
  private static final int LEGACY_ITERATIONS = 65_536;
  private static final int SALT_LENGTH = 16;
  private static final int HASH_KEY_LENGTH = 256;
  private static final String CURRENT_PREFIX = "pbkdf2-sha256$";

  public static PasswordVerifier hashPassword(String password) {
    byte[] salt = randomSalt();
    byte[] digest = derive(password, salt, CURRENT_ITERATIONS);
    return PasswordVerifier.current(CURRENT_PREFIX, CURRENT_ITERATIONS, salt, digest);
  }

  public static PasswordVerification verify(
      String password,
      String encodedVerifier,
      String legacySalt,
      String legacyHash
  ) {
    var parsed = PasswordVerifier.parseOrLegacy(
        encodedVerifier, legacySalt, legacyHash, LEGACY_ITERATIONS);
    byte[] candidate = derive(password, parsed.salt(), parsed.iterations());
    boolean matches = MessageDigest.isEqual(candidate, parsed.digest());
    return new PasswordVerification(matches, matches && parsed.needsUpgrade());
  }
```

Verification:

- `./gradlew.bat :cbell-lib:test :website:test --tests '*Password*Test' --tests '*AccountAuthenticationServiceTest' --no-daemon`

### Task 2 - Stable HTTP errors and validated client identity

Sequence / dependencies:

- Runs after Task 1 so authentication failure mapping uses the final public error contract.

Expected files or modules:

- `cbell-lib/src/main/java/dev/christopherbell/libs/api/controller/ControllerExceptionHandler.java`
- `website/src/main/java/dev/christopherbell/configuration/ClientIpProperties.java`
- `website/src/main/java/dev/christopherbell/configuration/ClientIpResolver.java`
- `website/src/main/resources/application-prod.yml`
- focused controller-advice, configuration-binding, rate-limit, and client-IP tests

Implementation notes:

- Required skill: `write-jane-street-style-code` before any code edits; execution must invoke it first.
- Before-Edit Brief:
  - Behavior: malformed public requests receive stable safe messages without ERROR stack traces; trusted forwarding works only for validated configured hops.
  - Invariants: unexpected 5xx failures retain causal ERROR logging; untrusted peers cannot influence client identity; invalid CIDRs stop startup.
  - Boundary/API: preserve existing response envelope codes and HTTP statuses while replacing raw exception descriptions.
  - Effects and failures: configuration binding owns CIDR parsing once at startup; request resolution performs no exception-swallowing fallback.
  - Tests and evidence: first add failing raw-message/log-level tests and invalid-IPv4/IPv6-CIDR binding tests; verify direct, trusted, and forged forwarding.

#### Code Edit 2.1

- File: `cbell-lib/src/main/java/dev/christopherbell/libs/api/controller/ControllerExceptionHandler.java`
- Lines: 46-58
- Action: replace

Current:

```java
  @ExceptionHandler(Exception.class)
  public ResponseEntity<Response<?>> handleGenericException(Exception e) {
    var frameworkStatus = statusForFrameworkException(e);
    if (frameworkStatus != null) {
      log.error(REQUEST_ERROR, e);
      return errorResponse(REQUEST_ERROR, e.getMessage(), frameworkStatus);
    }

    log.error(INTERNAL_SERVER_ERROR, e);
    return errorResponse(
        INTERNAL_SERVER_ERROR,
        "An unexpected error occurred. Please try again later.",
        HttpStatus.INTERNAL_SERVER_ERROR);
  }
```

Proposed:

```java
  @ExceptionHandler(Exception.class)
  public ResponseEntity<Response<?>> handleGenericException(Exception failure) {
    var frameworkStatus = statusForFrameworkException(failure);
    if (frameworkStatus != null) {
      log.warn("{} status={}", REQUEST_ERROR, frameworkStatus.value());
      return errorResponse(REQUEST_ERROR, "The request could not be processed.", frameworkStatus);
    }

    log.error(INTERNAL_SERVER_ERROR, failure);
    return errorResponse(
        INTERNAL_SERVER_ERROR,
        "An unexpected error occurred. Please try again later.",
        HttpStatus.INTERNAL_SERVER_ERROR);
  }
```

Verification:

- `./gradlew.bat :cbell-lib:test :website:test --tests '*ControllerExceptionHandlerTest' --tests '*ClientIp*Test' --no-daemon`

#### Code Edit 2.2

- File: `website/src/main/java/dev/christopherbell/configuration/ClientIpProperties.java`
- Lines: 11-15
- Action: replace

Current:

```java
@ConfigurationProperties(prefix = "client-ip")
@Data
public class ClientIpProperties {
  private List<String> trustedProxies = new ArrayList<>();
}
```

Proposed:

```java
@ConfigurationProperties(prefix = "client-ip")
@Validated
@Data
public class ClientIpProperties {
  private List<@TrustedProxyCidr String> trustedProxies = new ArrayList<>();

  public List<IpNetwork> parsedTrustedProxies() {
    return trustedProxies.stream().map(IpNetwork::parse).toList();
  }
}
```

Verification:

- `./gradlew.bat :website:test --tests '*ClientIpPropertiesTest' --tests '*ClientIpResolverTest' --no-daemon`

### Task 3 - WFL creator authorization, bounded membership, and safe websites

Sequence / dependencies:

- Runs after Task 2 because capacity and authorization denials use the stable public error contract.

Expected files or modules:

- `website/src/main/java/dev/christopherbell/whatsforlunch/restaurant/**`
- `website/src/main/resources/static/js/whats-for-lunch.js`
- `website/src/main/resources/static/js/restaurant-profile.js`
- WFL service/controller/import and JavaScript tests

Implementation notes:

- Required skill: `write-jane-street-style-code` before any code edits; execution must invoke it first.
- Before-Edit Brief:
  - Behavior: only the creator replaces restaurants/clears votes; total membership never exceeds 21; unsafe website schemes are rejected on write and suppressed on read.
  - Invariants: creator is always a participant; joins are idempotent; concurrency cannot exceed the cap; only absolute HTTP(S) URLs become active links.
  - Boundary/API: add a creator capability to session detail; keep public session links and valid restaurant records compatible.
  - Effects and failures: membership update must be atomic or optimistic with a stable capacity failure; imports report invalid URLs without partial unsafe persistence.
  - Tests and evidence: first add participant-denial, creator-capability, 21/22-member, concurrent-join, import/admin URL, and legacy-render tests.

#### Code Edit 3.1

- File: `website/src/main/java/dev/christopherbell/whatsforlunch/restaurant/session/WhatsForLunchSessionService.java`
- Lines: 108-127
- Action: replace

Current:

```java
  public WhatsForLunchSessionDetail joinSession(String sessionId)
      throws InvalidRequestException, ResourceNotFoundException {
    var self = getSelfAccount();
    var session = getSessionById(sessionId);
    var participantIds = new ArrayList<>(session.getParticipantAccountIds() == null
        ? List.of()
        : session.getParticipantAccountIds());
    var usernamesByAccountId = new LinkedHashMap<>(session.getParticipantUsernamesByAccountId() == null
        ? Map.<String, String>of()
        : session.getParticipantUsernamesByAccountId());

    if (!participantIds.contains(self.getId())) {
      participantIds.add(self.getId());
      usernamesByAccountId.put(self.getId(), self.getUsername());
      session.setParticipantAccountIds(List.copyOf(participantIds));
      session.setParticipantUsernamesByAccountId(usernamesByAccountId);
      session.setLastUpdatedOn(Instant.now());
      session = sessionRepository.save(session);
    }
    return toDetail(session, self.getId());
  }
```

Proposed:

```java
  public WhatsForLunchSessionDetail joinSession(String sessionId)
      throws InvalidRequestException, ResourceNotFoundException {
    var self = getSelfAccount();
    var outcome = sessionMemberships.joinIfCapacityRemains(
        sessionId, self.getId(), self.getUsername(), MAX_PARTICIPANTS);
    if (outcome == SessionJoinOutcome.FULL) {
      throw new InvalidRequestException("This WFL session has reached its member limit.");
    }
    return toDetail(getSessionForParticipant(sessionId, self.getId()), self.getId());
  }
```

Verification:

- `./gradlew.bat :website:test --tests '*WhatsForLunchSession*Test' --tests '*Restaurant*Test' --no-daemon`

#### Code Edit 3.2

- File: `website/src/main/java/dev/christopherbell/whatsforlunch/restaurant/session/WhatsForLunchSessionService.java`
- Lines: 156-167
- Action: replace

Current:

```java
  public WhatsForLunchSessionDetail updateRestaurants(
      String sessionId,
      WhatsForLunchSessionRestaurantsRequest request
  ) throws InvalidRequestException, ResourceNotFoundException {
    var self = getSelfAccount();
    var restaurantIds = normalizeRestaurantIds(request == null ? null : request.restaurantIds());
    var restaurants = getRestaurantsInRequestedOrder(restaurantIds);
    var session = getSessionForParticipant(sessionId, self.getId());
    session.setRestaurantIds(restaurantIds);
    session.setVotesByAccountId(new LinkedHashMap<>());
    session.setLastUpdatedOn(Instant.now());
    return toDetail(sessionRepository.save(session), self.getId(), restaurants);
  }
```

Proposed:

```java
  public WhatsForLunchSessionDetail updateRestaurants(
      String sessionId,
      WhatsForLunchSessionRestaurantsRequest request
  ) throws InvalidRequestException, ResourceNotFoundException {
    var self = getSelfAccount();
    var session = getSessionForParticipant(sessionId, self.getId());
    requireCreator(session, self.getId());
    var restaurantIds = normalizeRestaurantIds(request == null ? null : request.restaurantIds());
    var restaurants = getRestaurantsInRequestedOrder(restaurantIds);
    session.setRestaurantIds(restaurantIds);
    session.setVotesByAccountId(new LinkedHashMap<>());
    session.setLastUpdatedOn(Instant.now());
    return toDetail(sessionRepository.save(session), self.getId(), restaurants);
  }
```

Verification:

- `node --test website/src/test/js/wfl-freshness.test.js website/src/test/js/public-content.test.js && ./gradlew.bat :website:test --tests '*WhatsForLunchSession*Test' --no-daemon`

### Task 4 - DNS-bound link previews and safe preview image URLs

Sequence / dependencies:

- Runs independently after Task 2; it reuses the proven federation pinned-address transport pattern without changing federation code.

Expected files or modules:

- `website/src/main/java/dev/christopherbell/post/preview/**`
- `website/src/main/resources/static/js/lib/feed-render.js`
- `website/src/main/resources/static/js/lib/image-lightbox.js`
- focused destination-policy, HTTP transport, parser, and renderer tests

Implementation notes:

- Required skill: `write-jane-street-style-code` before any code edits; execution must invoke it first.
- Before-Edit Brief:
  - Behavior: every initial and redirect connection uses an address from the exact validated DNS answer set while preserving original Host, SNI, and TLS hostname verification; preview images are absolute HTTP(S) only.
  - Invariants: no second unbound DNS resolution, redirects never bypass destination policy, body/time/redirect bounds remain, and non-HTTP(S) image values never become clickable fallbacks.
  - Boundary/API: keep `FetchedPage` and preview-service behavior compatible; replace the transport seam behind it.
  - Effects and failures: create one fresh bounded connection per hop, no proxy/redirect/client retry, deterministic rejection of unsafe or mixed DNS answers.
  - Tests and evidence: first add a DNS answer-change transport test, Host/SNI assertions, unsafe redirect tests, and Java/JS non-HTTP image tests.

#### Code Edit 4.1

- File: `website/src/main/java/dev/christopherbell/post/preview/BoundedLinkPreviewHttpClient.java`
- Lines: 47-62
- Action: replace

Current:

```java
  public FetchedPage fetch(URI initialUri) {
    var deadlineNanos = System.nanoTime() + properties.getOverallTimeout().toNanos();
    var current = initialUri;
    for (var redirects = 0; ; redirects++) {
      requirePublic(current, deadlineNanos);
      var remaining = remaining(deadlineNanos);
      var request = HttpRequest.newBuilder(current)
          .GET()
          .timeout(shorter(properties.getRequestTimeout(), remaining))
          .header("Accept", "text/html, application/xhtml+xml")
          .header("User-Agent", "christopherbell.dev link preview fetcher")
          .build();
      final HttpResponse<InputStream> response;
      try {
        response = httpClient.send(request, HttpResponse.BodyHandlers.ofInputStream());
```

Proposed:

```java
  public FetchedPage fetch(URI initialUri) {
    var deadline = LinkPreviewDeadline.start(properties.getOverallTimeout());
    var current = initialUri;
    for (var redirects = 0; ; redirects++) {
      var destination = destinationPolicy.resolveApproved(current, deadline.remaining());
      var response = transport.get(
          destination,
          shorter(properties.getRequestTimeout(), deadline.remaining()),
          Map.of(
              "Accept", "text/html, application/xhtml+xml",
              "User-Agent", "christopherbell.dev link preview fetcher"));
```

Verification:

- `./gradlew.bat :website:test --tests '*PostLinkPreview*Test' --tests '*BoundedLinkPreviewHttpClientTest' --no-daemon && node --test website/src/test/js/image-lightbox.test.js website/src/test/js/feed-render-lifespan.test.js`

### Task 5 - Account-scoped upload resume state

Sequence / dependencies:

- Runs independently after Task 1 because it consumes the authoritative non-secret current account identifier.

Expected files or modules:

- `website/src/main/resources/static/js/shared-folder.js`
- shared-folder page bootstrap/account context
- shared-folder JavaScript tests

Implementation notes:

- Required skill: `write-jane-street-style-code` before any code edits; execution must invoke it first.
- Before-Edit Brief:
  - Behavior: each account reads and writes only its own resume record; legacy global state is removed without being displayed; terminal completion/discard clears the scoped record.
  - Invariants: the account identifier is not a secret or authorization control; server upload ownership remains authoritative; unauthenticated/missing identity reads no resume state.
  - Boundary/API: preserve the resume record schema and server upload endpoints; change only client key derivation/lifecycle.
  - Effects and failures: localStorage failure remains non-fatal; account switch cannot reuse another account's key.
  - Tests and evidence: first add two-account shared-browser tests, legacy-key non-read/removal, and terminal cleanup assertions.

#### Code Edit 5.1

- File: `website/src/main/resources/static/js/shared-folder.js`
- Lines: 45-48
- Action: replace

Current:

```javascript
const root = typeof document === 'undefined' ? null : document.getElementById('shared-folder-app');
let currentPreviewLostAccess = false;
const UPLOAD_RESUME_KEY = 'shared-folder-upload-resume-v1';
const uploadOperationGate = createUploadOperationGate();
```

Proposed:

```javascript
const root = typeof document === 'undefined' ? null : document.getElementById('shared-folder-app');
let currentPreviewLostAccess = false;
const LEGACY_UPLOAD_RESUME_KEY = 'shared-folder-upload-resume-v1';
const currentAccountId = root?.dataset.accountId || '';
const uploadResumeKey = currentAccountId
  ? `shared-folder-upload-resume-v2:${currentAccountId}`
  : null;
const uploadOperationGate = createUploadOperationGate();
```

Verification:

- `node --test website/src/test/js/shared-folder.test.js website/src/test/js/shared-folder-page-initialization.test.js website/src/test/js/shared-folder-streaming.test.js`

### Task 6 - Federation approval, per-post eligibility, and affirmative enrollment

Sequence / dependencies:

- Runs after Task 1 because it uses the same authoritative approved/active account state.

Expected files or modules:

- `website/src/main/java/dev/christopherbell/federation/**`
- `website/src/main/java/dev/christopherbell/account/AccountRepository.java`
- `website/src/main/resources/templates/signup.html`
- federation discovery, outbox, consent, publication, delivery, view, DTO, and signup JavaScript tests

Implementation notes:

- Required skill: `write-jane-street-style-code` before any code edits; execution must invoke it first.
- Before-Edit Brief:
  - Behavior: false/null per-post eligibility never appears in public outboxes; unapproved accounts cannot enroll, be discovered, create eligible posts, or deliver; signup defaults unchecked.
  - Invariants: current consent and identity remain necessary but not sufficient; missing approval fails closed; explicit user opt-in remains round-trippable; existing transport/signing behavior is unchanged.
  - Boundary/API: preserve ActivityPub response formats and routes; narrow only eligible records and accounts.
  - Effects and failures: Mongo count and page predicates stay identical; moderation takes effect on the next discovery/delivery state check.
  - Tests and evidence: first add false/null outbox exclusions, explicit unapproved fixtures at every federation boundary, and unchecked signup rendering/submission tests.

#### Code Edit 6.1

- File: `website/src/main/java/dev/christopherbell/federation/discovery/FederationOutboxQueryRepository.java`
- Lines: 55-58
- Action: replace

Current:

```java
  private static Criteria activeOwned(String accountId, Instant now) {
    return Criteria.where("accountId").is(accountId)
        .and("expiresOn").gt(now)
        .and("createdOn").ne(null);
  }
```

Proposed:

```java
  private static Criteria activeOwned(String accountId, Instant now) {
    return Criteria.where("accountId").is(accountId)
        .and("federationOutboundEligible").is(true)
        .and("expiresOn").gt(now)
        .and("createdOn").ne(null);
  }
```

Verification:

- `./gradlew.bat :website:test --tests '*FederationOutboxQueryRepositoryTest' --tests '*FederationCollectionServiceTest' --no-daemon`

#### Code Edit 6.2

- File: `website/src/main/java/dev/christopherbell/federation/outbound/FederationPublicationPolicy.java`
- Lines: 18-23
- Action: replace

Current:

```java
  public boolean eligibleAtCreation(Account account) {
    return account != null
        && properties.outboundEnabled()
        && account.getStatus() == AccountStatus.ACTIVE
        && account.isFederationEnabled()
        && account.getFederationIdentity() != null;
  }
```

Proposed:

```java
  public boolean eligibleAtCreation(Account account) {
    return account != null
        && properties.outboundEnabled()
        && account.getStatus() == AccountStatus.ACTIVE
        && Boolean.TRUE.equals(account.getIsApproved())
        && account.isFederationEnabled()
        && account.getFederationIdentity() != null;
  }
```

Verification:

- `./gradlew.bat :website:test --tests '*FederationDiscoveryServiceTest' --tests '*FederationConsentServiceTest' --tests '*FederationPublicationPolicyTest' --tests '*FederationOutboundCoordinatorTest' --no-daemon && node --test website/src/test/js/signup-auth.test.js`

#### Code Edit 6.3

- File: `website/src/main/resources/templates/signup.html`
- Lines: 48-64
- Action: replace

Current:

```html
<input
  class="form-check-input"
  id="federatePublicVoidPosts"
  type="checkbox"
  th:checked="${federationEnrollmentAvailable}"
  th:disabled="${!federationEnrollmentAvailable}" />
<p
  id="federationConsentStatus"
  class="form-text federation-consent-status"
  th:text="${federationEnrollmentAvailable} ? 'This choice is on by default. You can turn it off now or later from Profile.' : 'Federation enrollment is not available on this server right now.'">
  Federation enrollment status
</p>
```

Proposed:

```html
<input
  class="form-check-input"
  id="federatePublicVoidPosts"
  type="checkbox"
  th:disabled="${!federationEnrollmentAvailable}" />
<p
  id="federationConsentStatus"
  class="form-text federation-consent-status"
  th:text="${federationEnrollmentAvailable} ? 'This choice is off until you explicitly enable it. You can change it later from Profile.' : 'Federation enrollment is not available on this server right now.'">
  Federation enrollment status
</p>
```

Verification:

- `./gradlew.bat :website:test --tests '*ViewControllerTest' --tests '*AccountCreateRequestFederationConsentTest' --no-daemon && node --test website/src/test/js/signup-auth.test.js website/src/test/js/federation-consent.test.js`

### Task 7 - Immutable build and workflow inputs

Sequence / dependencies:

- Runs after behavioral changes so verification metadata covers the final resolved build graph.

Expected files or modules:

- `.github/workflows/*.yml`
- `gradle/wrapper/gradle-wrapper.properties`
- new `gradle/verification-metadata.xml`
- build documentation and security configuration tests

Implementation notes:

- Required skill: `write-jane-street-style-code` before editing workflows, executable build configuration, or reusable verification automation; execution must invoke it first.
- Before-Edit Brief:
  - Behavior: wrapper, plugins, dependencies, and Actions execute only reviewed immutable bytes.
  - Invariants: exact Gradle 9.6.1 distribution remains; intended Action releases remain visible in comments; strict verification is the default.
  - Boundary/API: no application runtime API change; contributor metadata updates follow a documented explicit command and review.
  - Effects and failures: unknown artifacts or checksum drift fail the build/CI; metadata must cover all tasks used by CI, packaging, CodeQL, and sensors.
  - Tests and evidence: first run repository config checks that fail on mutable tags/missing wrapper sum/missing metadata, then generate/review metadata and verify from a clean isolated Gradle home.

#### Code Edit 7.1

- File: `gradle/wrapper/gradle-wrapper.properties`
- Lines: 1-9
- Action: replace

Current:

```properties
distributionBase=GRADLE_USER_HOME
distributionPath=wrapper/dists
distributionUrl=https\://services.gradle.org/distributions/gradle-9.6.1-bin.zip
networkTimeout=10000
retries=0
retryBackOffMs=500
validateDistributionUrl=true
zipStoreBase=GRADLE_USER_HOME
zipStorePath=wrapper/dists
```

Proposed:

```properties
distributionBase=GRADLE_USER_HOME
distributionPath=wrapper/dists
distributionSha256Sum=<authoritative-gradle-9.6.1-bin-sha256>
distributionUrl=https\://services.gradle.org/distributions/gradle-9.6.1-bin.zip
networkTimeout=10000
retries=0
retryBackOffMs=500
validateDistributionUrl=true
zipStoreBase=GRADLE_USER_HOME
zipStorePath=wrapper/dists
```

Verification:

- `$env:GRADLE_USER_HOME = (Join-Path $env:TEMP 'christopherbell-security-gradle'); ./gradlew.bat --version; ./gradlew.bat :website:check --dependency-verification=strict --no-daemon`

#### Code Edit 7.2

- File: `.github/workflows/ci.yml`
- Lines: 22-62
- Action: replace

Current:

```yaml
- uses: actions/checkout@v4
- uses: actions/setup-java@v4
```

Proposed:

```yaml
- uses: actions/checkout@<reviewed-full-commit-sha> # v4
- uses: actions/setup-java@<reviewed-full-commit-sha> # v4
```

Verification:

- `rg -n 'uses: [^#\r\n]+@v[0-9]' .github/workflows; ./gradlew.bat :website:check --dependency-verification=strict --no-daemon`

### Task 8 - Integrated verification, delivery, and production acceptance

Sequence / dependencies:

- Runs after Tasks 1-7 are green and committed in reviewable spoke commits.

Implementation notes:

- No code edit is planned in this task.
- Re-run a final security diff review over the complete branch.
- Use an isolated `GRADLE_USER_HOME`; run `:website:check`, strict verification, JavaScript syntax/tests, and the packaged production-profile JAR.
- Start the exact artifact on a non-8080 port, exercise anonymous, login, bearer revocation, proxy, WFL, unsafe URL, link-preview, upload-state, and federation boundaries, then stop the alternate listener.
- Save the Builder test report and spoke review, open a PR, wait for CI/Dependency Review/CodeQL, address only trusted `azurras` guidance, merge when green, close/update relevant issues, deploy through existing guarded automation, and verify listener, root smoke, service, migration/index, and external reachability.

Verification:

- `./gradlew.bat :website:check --dependency-verification=strict --no-daemon`
- `node --test website/src/test/js/*.test.js`
- production-profile alternate-port and post-deploy smoke commands from `verify-local-spring-app`

## Code Changes

- Authentication/password: Code Edits 1.1-1.2.
- Error and proxy boundary: Code Edits 2.1-2.2.
- WFL authorization/capacity/URLs: Code Edits 3.1-3.2 plus shared HTTP(S) URL parsing described in Task 3.
- Link-preview SSRF and image URLs: Code Edit 4.1 plus Java/JS image scheme guards described in Task 4.
- Browser upload resume isolation: Code Edit 5.1.
- Federation privacy/moderation: Code Edits 6.1-6.3.
- Build supply chain: Code Edits 7.1-7.2 plus generated reviewed verification metadata.

## Files and Modules

- `cbell-lib` security and API error utilities.
- Website account, authentication, configuration, WFL, post-preview, shared-folder, and federation modules.
- Website templates/static JavaScript and corresponding JavaScript tests.
- Gradle wrapper/verification metadata and GitHub workflows.
- Builder spec, plan, test report, spoke review, issue closure, session memory, and work closure artifacts.

## Unit Testing

- Every task begins with a failing regression at the narrowest security boundary and records the expected RED reason before production code.
- Run focused Gradle tests with `--tests` after each Java edit group.
- Run focused `node --test` suites and `node --check` for changed JavaScript.
- Add concurrency evidence for WFL capacity and address-pinning evidence for link previews.

## Local Testing

- Use an isolated Gradle home to avoid Windows registry/file locks.
- Run the full `:website:check` and strict dependency verification.
- Package and run the production profile on a non-8080 port.
- Exercise representative HTTP success and failure flows with no live-listener changes.
- Verify Mongo migrations/indexes needed by the resulting artifact.

## Validation

- All 17 reportable findings map to a code/config fix and regression evidence.
- The self-only signup privacy hardening is tested and not misrepresented as a reportable vulnerability.
- The final branch security diff has no surviving reportable finding.
- Full automated, alternate-port, CI, merged-main, and production checks pass.

## Rollback or Recovery

- Keep boundary changes in cohesive spoke commits so an individual regression can be reverted without discarding unrelated fixes.
- Legacy password verification remains available until successful migration; rollback preserves stored legacy fields.
- WFL membership and federation changes require no destructive data migration.
- Dependency metadata and Action pins can be reverted as one supply-chain commit if an upstream artifact mismatch is proven, never bypassed in place.
- Do not rotate the live service until the exact packaged artifact passes alternate-port checks; use the existing guarded deployment rollback if post-cutover smoke fails.

## Risks

- PBKDF2 cost can increase login CPU latency; benchmark representative verification and keep it under one second on the production host.
- Bearer account reload adds one database read; preserve the indexed ID lookup and fail closed on repository failure.
- A naive WFL read-then-save cap races; use atomic conditional update or optimistic locking with concurrency tests.
- Hostname-preserving IP pinning is easy to break at TLS/SNI; use the federation transport as a negative/positive control and test exact connection address.
- Strict Gradle metadata can miss rarely resolved configurations; generate against the final full task set and verify from a clean home.
- Production shares the development host; alternate-port validation and guarded listener rotation are mandatory.

## Completion Criteria

- Plan tasks 1-7 have RED/GREEN evidence and reviewable commits.
- Focused and full checks pass with zero failures, strict dependency verification enabled.
- Packaged alternate-port production-profile smoke passes before deployment.
- PR checks and CodeQL are green, PR merged, relevant issues closed/updated with evidence.
- Production listener/service/endpoints/migrations/external reachability pass without ACL weakening.
- Builder test report, spoke review, closure, session memory, indexes, validation, and commits are complete.
