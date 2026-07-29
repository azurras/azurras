# ChristopherBell.dev ActivityPub Production Discovery Activation

## Document Status

Ready for execution. The user authorized autonomous continuation, and the recommended design is selected.

## Purpose

Activate the existing read-only ActivityPub discovery surface on production through the ordinary push-to-main deployment path. The activation must be automatic, noninteractive, reversible, and unable to enable inbound mutations or uncontrolled outbound delivery.

## Background

The discovery foundation and controlled outbound delivery implementation are merged and production-verified, but all federation switches remain disabled. Production runs as `LocalSystem`; its `C:\ProgramData\christopherbell.dev\config` directory is already restricted to SYSTEM and Administrators. Automatic deployment builds a candidate from `origin/main`, starts it with the production profile on port 8081, switches the release junction, and verifies the public site. It intentionally does not rewrite the protected `app.env` file or refresh installed service scripts on every release.

No real operator-controlled ActivityPub peer inbox is present in the repository or Builder records. Outbound activation therefore cannot be performed honestly in this gate.

## Goals

- Publish NodeInfo, WebFinger, actor, outbox, followers, and following discovery for explicitly consented accounts.
- Generate a dedicated 256-bit federation identity-encryption secret exactly once when production discovery first starts.
- Reuse the same secret across candidates, deployments, restarts, and rollbacks.
- Store the secret only under the existing protected production config directory and never log its value.
- Make candidate, local-production, and public deployment checks prove that discovery is live.
- Preserve noninteractive push-to-main deployment.
- Keep inbound and outbound federation disabled by default and in production.

## Non-Goals

- No inbound ActivityPub POST handling.
- No uncontrolled outbound delivery.
- No arbitrary or third-party peer destination.
- No federation of historical posts.
- No change to account consent semantics.
- No secret service, cloud credential, DNS change, or external infrastructure provisioning.
- No administrative UI for federation configuration in this gate.

## Considered Approaches

### Manual protected environment update

Add federation variables to `app.env` and require an elevated restart. This is mechanically simple but conflicts with automatic deployment, creates recurring UAC/manual failure modes, and cannot update an existing protected file from an ordinary source change.

### Derive from an existing application secret

Derive the federation encryption key from `APP_JWT_SECRET`. This avoids a new file, but it couples unrelated security domains. Rotating the JWT secret would make every stored federation private key unreadable. This approach is rejected.

### Dedicated atomically created production secret

When and only when the production profile has discovery enabled and no explicit secret override is supplied, resolve a configured secret file under the protected production config directory. Read a valid existing 32-byte secret or create one with a cryptographically secure generator and create-new semantics. Inject only the encoded value into the application environment before federation configuration binds. This approach is selected.

## Proposed Approach

### Startup secret boundary

Add a focused production initializer responsible only for resolving the federation encryption secret before Spring binds `FederationProperties`.

The initializer:

1. Does nothing outside the production profile.
2. Does nothing when discovery is disabled.
3. Honors an explicit `APP_FEDERATION_KEY_ENCRYPTION_SECRET` override without touching disk.
4. Requires a configured secret-file path when production discovery is enabled without an override.
5. Rejects symlinks/reparse-style symbolic paths and non-regular files.
6. Reads exactly 32 bytes from an existing file.
7. Otherwise creates 32 cryptographically random bytes atomically with create-new and no-follow semantics.
8. Handles a concurrent create by reading and validating the winning file.
9. Adds a highest-precedence non-enumerable property source for the base64 value without logging it.
10. Clears temporary byte arrays after use where practical.

The production default file is `C:/ProgramData/christopherbell.dev/config/federation-key-encryption-secret.bin`. Its parent is already protected by the production installer and owned by SYSTEM/Administrators. The file contains raw random bytes, not a checked-in placeholder.

### Activation switches

`application-prod.yml` defaults discovery to enabled while retaining an environment kill switch:

- `APP_FEDERATION_DISCOVERY_ENABLED` defaults to `true`.
- `APP_FEDERATION_INBOUND_ENABLED` defaults to `false`.
- `APP_FEDERATION_OUTBOUND_ENABLED` defaults to `false`.
- The controlled peer list remains empty.
- Development loopback remains false.

An operator can immediately disable discovery by setting `APP_FEDERATION_DISCOVERY_ENABLED=false` through the protected environment boundary. Disabling discovery avoids secret resolution and returns the existing indistinguishable 404 responses.

### Deployment proof

Add `/.well-known/nodeinfo` and `/nodeinfo/2.1` to the existing candidate and public smoke path set. A deployment cannot become current when discovery failed to initialize or those public documents are unavailable.

WebFinger and actor checks remain post-deploy acceptance checks because they require a known explicitly consented account. The generic deployment smoke path must not assume a username or create consent.

### Outbound readiness

Outbound remains disabled and peerless. Its existing startup invariant continues to require discovery, a not-before cutoff, and at least one bounded controlled peer. A later gate may supply a real operator-controlled inbox through an explicit, reviewed configuration mechanism and then enable outbound.

## Security Invariants

- No real secret is committed, printed, returned, or placed in release metadata.
- Missing, malformed, inaccessible, symlinked, or wrong-sized secret storage fails startup before federation services bind.
- Secret creation never truncates or replaces an existing file.
- The same secret survives deployments and rollbacks.
- Discovery activation cannot imply inbound or outbound activation.
- Public discovery exposes only the already-bounded consented-account projections.
- Deployment rollback remains available if candidate or public discovery checks fail.

## Files and Modules

Expected spoke changes:

- `website/src/main/java/dev/christopherbell/configuration/` for the startup resolver/initializer.
- `website/src/main/resources/META-INF/spring.factories` for initializer registration.
- `website/src/main/resources/application-prod.yml` for the discovery default and secret-file path.
- `website/src/test/java/dev/christopherbell/configuration/` for startup and production configuration behavior.
- `ops/production/windows/modules/Production.Deploy.psm1` and its Pester tests for discovery smoke paths.
- `website/src/main/java/dev/christopherbell/federation/README.md` and `docs/operations/windows-production.md` for the activation and rollback contract.

## Validation Plan

- RED/GREEN unit coverage for new secret creation, stable reuse, explicit override, disabled/non-production no-op, malformed file rejection, and concurrent-create recovery.
- Configuration test proving production discovery defaults on while inbound/outbound remain off and peerless.
- Pester test proving both NodeInfo routes are in candidate and public smoke checks.
- Focused Java and PowerShell tests.
- Full `:website:check` regression.
- Alternate-port production-profile startup using an isolated Mongo database and isolated secret file.
- Verify NodeInfo 200, foreign/missing WebFinger 404, inbound POST denied, no outbound jobs, and no remote requests.
- One PR with green CI and CodeQL, merge to main, automatic deployment, then local/public production acceptance.

## Acceptance Criteria

- A push to main deploys without a terminal, confirmation prompt, or UAC interaction.
- Production `/.well-known/nodeinfo` and `/nodeinfo/2.1` return 200 with no-store behavior.
- Production startup creates or reuses one protected 32-byte federation secret without exposing it.
- Restart and redeploy preserve federation identity decryptability.
- Inbound and outbound remain disabled; no delivery job or remote call is created by activation.
- The normal site and existing authenticated features remain healthy.

## Open Questions

None block this gate. A real operator-controlled peer inbox is required before a later outbound-activation gate can proceed.
