# ChristopherBell.dev ActivityPub Controlled Outbound Delivery Test Report

## Document Status

complete

## Story/Issue

Second ActivityPub rollout gate from the approved Void public-growth program: controlled outbound delivery. Plan: `docs/implementation-plans/2026-07-28-christopherbell-dev-activitypub-controlled-outbound-delivery.md`.

## Branch

- Spoke: `azurras/christopherbell.dev`
- Branch: `codex/activitypub-outbound-delivery`
- Final branch commit: `2ea0ddb8`
- PR: `azurras/christopherbell.dev#1317`
- Merged main SHA: `6c1501070ff518bc040583c4576c2df201dcd3ed`

## App / Environment

- ChristopherBell.dev Spring Boot 4.1.0 on Java 25.
- Worktree: `A:\Projects\christopherbell.dev-worktrees\activitypub-outbound-delivery`.
- Enabled candidate: local profile, port 8081, base URL `http://127.0.0.1:8081`.
- Controlled peer: loopback port 18082, inbox `/inbox`.
- MongoDB: `mongodb://127.0.0.1:27017`, isolated database `christopherbell_activitypub_outbound_test_20260729`.
- Production: native Windows service on port 8080 and `https://www.christopherbell.dev/`.

## Local Run Details

The enabled candidate ran the boot JAR with local profile, port 8081, the isolated database, public base URL `http://127.0.0.1:8081`, discovery and outbound enabled, inbound disabled, a generated 32-byte test encryption secret, not-before `2026-07-28T00:00:00Z`, peer `http://127.0.0.1:18082/inbox`, the explicit development-loopback flag, one-second initial backoff, five-second maximum backoff, three attempts, and 250 ms scan/delivery delays. Mail was disabled.

The peer returned 503 plus `Retry-After: 1` once, then 202. Enabled app PID 41712, disabled app PID 45292, and peer PID 46312 were stopped after testing. Ports 8081 and 18082 were confirmed free. The isolated database and temporary fixture/log were deleted. Production was never manually stopped.

## Test Cases

1. Enabled startup and migration recovery: started the full app with discovery/outbound enabled. Runtime found and drove fixes for a final Mongo repository that Spring could not proxy and a Jackson 2 mapper dependency in this Jackson 3 application. Focused context tests now reproduce both failures. PASS after correction.
2. Signed retry: created a new explicitly opted-in account and public Void post. The peer returned 503, then 202. PASS: exactly two signed, digested requests used the same activity ID and identical body hash; the job ended SUCCEEDED after two attempts.
3. Kill switch: restarted without a federation secret or peer and with discovery/inbound/outbound false, then created another post. PASS: eligibility false, no job, and no new peer request.
4. Full verification: ran `A:\Temp\gradle-activitypub-outbound` as isolated Gradle home and `gradlew.bat :website:check --no-daemon --console=plain`. PASS: 201 suites, 1,382 tests, zero failures/errors, boot JAR and sensor runtime verification complete.
5. CI/deployment: PR 1317 passed Windows, Ubuntu, macOS, dependency review, and CodeQL, then rebased into main. The hidden SYSTEM auto-deployer rotated the live listener from PID 33352 to 16956. PASS: local/public home 200, federation discovery 404, migration/index state correct, zero production jobs or eligible posts.

## Data Sent

- `GET http://127.0.0.1:8081/signup` to obtain CSRF state.
- `POST /api/accounts/2024-12-15/create` with `Federation Runtime`, username `federation-runtime-20260729`, test email `federation-runtime-20260729@example.test`, redacted password, and `federatePublicVoidPosts: true`.
- `POST /api/accounts/2024-12-15/login` with the test email and redacted password.
- Authenticated `POST /api/posts/2025-09-14/create` with text `Controlled ActivityPub outbound runtime acceptance`.
- After disabled restart, authenticated POST of `Federation kill switch runtime acceptance`.
- Read-only production GET requests to `/`, `/.well-known/nodeinfo`, `/nodeinfo/2.1`, and `/.well-known/webfinger?...`, plus the public home page.
- Read-only Mongo queries for migration/index names, eligible-post count, and delivery-job count.

## Response Received

- Status code: 200 for signup and enabled candidate root.
- Status code: 503 for the first peer request with Retry-After 1; status code: 202 for the retry.
- Account ID: `ef0398d1-1ad1-4633-9b9b-202577dbab42`.
- Enabled post ID: `14f5f9fe-b81a-42bb-b52a-058035aeafeb`.
- Both requests used activity ID `http://127.0.0.1:8081/void/14f5f9fe-b81a-42bb-b52a-058035aeafeb#activity`; body hash matched; Signature and Digest were present.
- Durable job: SUCCEEDED, attempts 2, lastStatus 202, outcome DELIVERED.
- Disabled post ID: `68b61955-2b55-4426-95df-240555ed6231`; eligibility false; jobs zero; peer count stayed 2.
- Production status code: 200 locally and publicly for `/`; status code: 404 for NodeInfo and WebFinger.
- Production migration 007: APPLIED; eligible posts zero; jobs zero; all four indexes present.

## Pass / Fail

PASS. Controlled delivery, stable signed retry, durable state, kill switch, fail-closed production defaults, automatic deployment, and production no-effect state met the plan. No failed acceptance case remains.

## Evidence

- Spoke commits: `f5c99be0`, `7eb01b9f`, `eab9a9ea`, `2ea0ddb8`.
- PR: `https://github.com/azurras/christopherbell.dev/pull/1317`.
- Merge SHA: `6c1501070ff518bc040583c4576c2df201dcd3ed`.
- Local BUILD SUCCESSFUL in 1m44s; 1,382 tests, zero failures/errors.
- CI: Windows 6m27s, Ubuntu 1m44s, macOS 1m40s; dependency review and all CodeQL analyses passed.
- Migration checksum: `1f9d1ddf7fcdb35e66556310b541bedbed4444c2178ebdf0dc22455c22b9ec82`.
- Production listener rotation: 33352 to 16956; native services Running/Automatic.

## Bugs / Follow-ups

No defect remains in this gate. Production outbound is intentionally disabled. Production activation, Update/Delete lifecycle delivery, broader peer fanout/discovery, inbound follows, and signed/idempotent inbound keep-alives/replies remain separate later gates.
