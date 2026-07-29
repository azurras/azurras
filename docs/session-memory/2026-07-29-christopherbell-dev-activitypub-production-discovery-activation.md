# 2026-07-29 - ChristopherBell.dev ActivityPub Production Discovery Activation

## 07:30 - Production discovery activated and verified

### Request

Continue the approved Void public-growth work autonomously, minimize approval and review churn, commit/push completed work, and prioritize working behavior and security. This gate needed to activate production ActivityPub discovery without enabling inbound or uncontrolled outbound behavior.

### Project Context

Builder is the workflow hub at `C:\Users\Christopher\Developer\builder`; `azurras/christopherbell.dev` is the spoke. The authoritative spoke checkout is dirty and was preserved. Work used isolated worktree `A:\Projects\christopherbell.dev-worktrees\activitypub-production-activation` on `codex/activitypub-production-activation`. The native Windows production host uses a hidden SYSTEM auto-deployer, MongoDB, the WinSW website service, a media worker, and cloudflared.

### Work Completed

- Added a pre-binding initializer that creates or reuses one random 32-byte federation encryption secret with atomic `CREATE_NEW`, concurrent-creator recovery, strict file checks, redacted errors, and byte clearing.
- Pinned ordinary production to `C:\ProgramData\christopherbell.dev\config\federation-key-encryption-secret.bin`; only the `deploy-smoke` profile permits an alternate absolute path. Explicit `APP_FEDERATION_KEY_ENCRYPTION_SECRET` still wins.
- Enabled production discovery by default while leaving inbound/outbound false, peers empty, and loopback false.
- Added both NodeInfo routes to local candidate and every public-host deployment smoke matrix.
- Added eight initializer tests, hardened production configuration tests, updated Pester expectations, and updated federation/configuration/Windows operations documentation.
- Committed spoke change `1244dd31`, opened PR 1318, passed all checks, and rebased into main at `8405cd77d0f1743fe33d70cc80b47e37048090a0`.
- Automatic deployment rotated production from PID 16956 to 39760 without prompts or manual service changes.
- Saved the runtime test report, spoke review, work closure, updated program status, and closed the work record.

### Decisions

- Discovery is the only production capability enabled. No real controlled peer exists on this host, so outbound activation was not faked or broadened.
- A persistent host key is generated at startup rather than committed or placed in an ordinary environment file. The protected parent must already exist.
- Alternate key paths are a test-only capability because configurable production paths would weaken the file-trust boundary.
- The existing automatic deployment mechanism remains the sole deployment path; protected ACL denial from this non-elevated shell is expected and was not bypassed.

### Validation

- TDD witnessed missing-initializer and old deployment-route expectations fail before implementation, then pass afterward. A later security test proved arbitrary production paths were initially accepted, failed RED, and passed after pinning.
- Fresh final `:website:check`: 202 suites, 1,390 tests, zero failures/errors, three skipped; build completed successfully.
- Fresh Pester 5.9.0 deployment suite: 37 passed, zero failed/skipped.
- Alternate-port `prod,deploy-smoke` runtime on 8091 and isolated MongoDB: root and NodeInfo 200, foreign WebFinger 404, inbox POST 403, 32-byte key fingerprint unchanged across restart, zero jobs/accounts/posts. Exact processes stopped, port closed, and database dropped.
- PR CI: Windows, Ubuntu, macOS, dependency review, and Actions/Java/JavaScript CodeQL passed.
- Production: canonical/apex root and NodeInfo 200; foreign WebFinger 404; inbox POST 403; zero delivery jobs, opted-in accounts, identities, eligible posts, and scan state. MongoDB, website, media worker, and cloudflared Running/Automatic.

### Current State

- Production spoke main: `8405cd77d0f1743fe33d70cc80b47e37048090a0` is behaviorally confirmed by the newly live NodeInfo routes and listener rotation; the protected release file is unreadable to this shell.
- Production discovery is enabled. Inbound/outbound are disabled and have produced no effects.
- Feature worktree is clean and its remote branch was deleted after merge. Builder is on main.
- Evidence: `docs/test-reports/2026-07-29-activitypub-production-discovery-activation.md`, `docs/spoke-reviews/2026-07-29-christopherbell-dev-activitypub-production-discovery-activation-review.md`, and `docs/work-closures/2026-07-29-christopherbell-dev-activitypub-production-discovery-activation.md`.

### Follow-ups

No required action remains for discovery activation. NodeInfo software version `unknown` is cosmetic. Outbound production delivery awaits a real operator-controlled peer and separate interoperability proof; inbound follows and interactions remain later gates. Start future work from fresh `origin/main` in a new isolated worktree and preserve the production secret.
