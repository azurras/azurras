# 2026-07-29 - ChristopherBell.dev ActivityPub Controlled Outbound Delivery

## 00:53 - ChristopherBell.dev ActivityPub Controlled Outbound Delivery

### Request

Continue the approved Void public-growth program autonomously, keep commit/push checkpoints, avoid approval/review churn, prioritize working behavior and security, and complete the controlled ActivityPub outbound delivery gate as one coherent PR. Production had to remain disabled.

### Project Context

Builder is the workflow hub at `C:\Users\Christopher\Developer\builder`; `azurras/christopherbell.dev` is the spoke. Work used isolated worktree `A:\Projects\christopherbell.dev-worktrees\activitypub-outbound-delivery` on branch `codex/activitypub-outbound-delivery`; the authoritative spoke checkout was preserved. The native Windows production host runs MongoDB, ChristopherBellDev through WinSW, and Cloudflared, with a hidden SYSTEM auto-deployer following `origin/main`.

### Work Completed

- `f5c99be0`: added typed bounded outbound settings, controlled peer allowlist, strict URI/DNS/global-address policy, deterministic pinned Reactor Netty connection behavior, hostname verification, no redirects/proxy/implicit retry, timeouts, typed response classification, and real HTTP/TLS tests.
- `7eb01b9f`: added shared canonical ActivityPub Create/Note construction, exact-byte Digest and RSA-SHA256 HTTP signatures using account-bound encrypted keys, zeroization of decrypted PKCS#8 bytes, and signer/activity tests.
- `eab9a9ea`: added nullable-safe creation-time post eligibility, durable Mongo jobs/cursor, stable job IDs, idempotent enqueue, ascending scan, leased atomic claims, exact-owner CAS transitions, retry/backoff, current author/post/peer rechecks, scheduler/gateway wiring, migration V007, and coordinator/policy tests.
- `2ea0ddb8`: added production-safe configuration/docs, fixed two runtime-only startup defects (Spring could not proxy the final Mongo repository; outbound wiring requested Jackson 2 instead of the app's Jackson 3 mapper), and added focused proxy/configuration regression tests.
- PR `azurras/christopherbell.dev#1317` passed all checks. Merge commits are disabled, so it rebased into main at `6c1501070ff518bc040583c4576c2df201dcd3ed` with task commits preserved.
- The SYSTEM poller automatically deployed main; production listener rotated from PID 33352 to 16956 without a terminal or approval prompt.
- Updated the program release status and plan status to complete. Saved test report, spoke review, and closure records in Builder.

### Decisions

- Only newly created posts may be marked eligible; missing historical values are false and no backfill occurs.
- WRITE-like remote effects are owned by a conditional outbound graph that does not exist while disabled.
- Operator-controlled peers are the only destinations in this gate. Fresh DNS validation rejects the whole result if any address is unsafe, then pins one deterministic address while keeping original Host/SNI verification.
- Stable activity IDs and exact repeated body bytes provide receiver-visible retry idempotency; the local deterministic post/peer job ID prevents duplicate scheduling.
- Production stays discovery/inbound/outbound false, with empty peers and loopback false. Activation is separate authority and evidence.

### Validation

- Final local `:website:check`: 201 suites, 1,382 tests, zero failures/errors; boot JAR and sensor runtime verification passed.
- Enabled candidate on 8081 plus isolated DB and peer on 18082: new opted-in account/post produced signed, digested 503 then 202 requests with identical activity ID/body. Mongo job ended SUCCEEDED after two attempts, migration 007 APPLIED, all four indexes present.
- Disabled candidate with no key/peer: root 200, new post eligibility false, zero job, no additional peer request.
- PR and main CI: Windows, Ubuntu, macOS, dependency review, Actions/Java/JavaScript CodeQL all passed.
- Production: local/public `/` 200; WebFinger and NodeInfo 404; migration 007 APPLIED; all indexes present; eligible-post count zero; delivery-job count zero; MongoDB, ChristopherBellDev, and Cloudflared Running/Automatic.
- Enabled/disabled candidate processes, peer fixture/log, ports 8081/18082, and isolated database were cleaned up. Production was not manually stopped.

### Current State

- Spoke main deployed SHA: `6c1501070ff518bc040583c4576c2df201dcd3ed`.
- Production federation remains fully disabled and has performed zero deliveries.
- Spoke feature worktree is clean; its remote feature branch was deleted by merge. Builder is on main and will be committed/pushed after indexes/validation.
- Runtime report: `docs/test-reports/2026-07-29-christopherbell-dev-activitypub-controlled-outbound-delivery-test-report.md`.
- Review: `docs/spoke-reviews/2026-07-29-christopherbell-dev-activitypub-controlled-outbound-delivery-review.md`.
- Closure: `docs/work-closures/2026-07-29-christopherbell-dev-activitypub-controlled-outbound-delivery.md`.

### Follow-ups

No defect remains in this gate. A future explicitly authorized gate may activate outbound to a controlled real peer after interoperability evidence. Update/Delete propagation, broader peer discovery/fanout, inbound follows, and signed/idempotent inbound keep-alives/replies remain separate later gates. Start future work from fresh `origin/main` in a new isolated worktree.
