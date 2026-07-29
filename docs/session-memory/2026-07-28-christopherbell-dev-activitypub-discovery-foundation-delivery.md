# 2026-07-28 - ChristopherBell.dev ActivityPub Discovery Foundation Delivery

## 23:29 - ChristopherBell.dev ActivityPub Discovery Foundation Delivery

### Request

Continue autonomously from the approved Void public-growth program, commit and push completed tasks, avoid repeated approvals/review churn, and ship working software without security regressions. This turn completed Release 3 gate 1: consent-first ActivityPub discovery.

### Project Context

- Builder hub: `C:\Users\Christopher\Developer\builder`, branch `main`.
- Spoke: `azurras/christopherbell.dev`.
- Isolated worktree: `A:\Projects\christopherbell.dev-worktrees\activitypub-federation`, branch `codex/activitypub-federation`, based on Release 2 SHA `f77c5f5bb644cc75cf98b27e722efdc00cd036f1`.
- The authoritative checkout at `A:\Projects\christopherbell.dev` remained untouched and may contain unrelated user state.
- Production is native Windows on port 8080 with automatic SYSTEM deployment from `origin/main`; all pre-merge runtime checks used port 8081 and a separate database.
- User approval remains durable for continuing approved work without repeated pauses. Ask only for new authority, a material scope change, or a genuine unresolved blocker.

### Work Completed

PR `azurras/christopherbell.dev#1316` merged as `6cd9e397e4ec2c3175ae5c31a95f633b7a7c7c95` after three pushed task commits:

- `d7c11fa3`: added explicit nullable-safe account consent, stable per-account RSA-2048 identities, AES-256-GCM encryption with fresh nonce and account/actor/key/version AAD, conditional federation configuration, three fail-closed flags, separate protected secret validation, and migration V006.
- `50cf469a`: added read-only WebFinger, NodeInfo 2.1, Person actors, active-post outbox/ordered pages, followers/following, exact anonymous GET security matchers, uniform no-store/CORS/nosniff headers, and stable cursor limits. No HTTP client, remote persistence, delivery scheduler, or federation mutation was added.
- `45b3618b`: added authoritative federation status API, signup default-on only when enrollment is configured, disabled unavailable state, Profile opt-in/out control with server-confirmed rollback, privacy disclosure, and tests. An already-enrolled user can opt out even if new enrollment becomes unavailable.

The production default remains discovery/inbound/outbound false with no federation secret required. Messages, Music, Shared Folder, reports, and administrative data are never exposed.

### Decisions

- Reused `app.browser-security.public-base-url` as the sole actor origin to prevent split identities.
- Existing accounts and API clients that omit consent remain opted out. Browser default-on applies only when enrollment is operational.
- Disabling consent immediately removes discoverability but retains encrypted identity, so re-enable preserves the actor and key.
- Kept this as the discovery/metadata gate only. Full outbound/inbound work was not pulled forward.
- Used one PR and limited review effort to working behavior, interoperability, correctness, and security.
- Did not weaken protected production ACLs when `prod.cmd auto-status` was denied. Deployment was proved by listener rotation and version-specific response behavior.

### Validation

- Full `:website:check`: BUILD SUCCESSFUL; 1,353 Java tests, zero failures/errors, three skips; complete JavaScript and sensor/package runtime checks passed.
- PR checks: Ubuntu, macOS, Windows, Dependency Review, CodeQL Java/Kotlin, JavaScript/TypeScript, and Actions all passed.
- Enabled local runtime on port 8081/database `christopherbell_activitypub_discovery_test`: opted-in signup, WebFinger/JRD, NodeInfo, actor, empty outbox/page, followers/following, headers, negative discovery, and blocked inbox POST passed.
- Restart changed PID 44916 to 46752 while actor ID, key ID, public key, WebFinger subject, and self link stayed stable.
- Authenticated toggle: status GET was authoritative; disable returned 200 and actor 404; re-enable returned 200 and restored the exact same public key.
- Disabled runtime PID 39624 started with no encryption key; root returned 200, federation discovery returned 404, and signup rendered a disabled choice.
- Cleanup: stopped only port-8081 candidate processes, dropped only the isolated database, removed test logs/scripts, and confirmed no port-8081 listener.
- Automatic production deploy rotated 8080 from PID 34768 to PID 33352. Public/local home and signup returned 200; new consent markup was present but disabled; NodeInfo/actor returned 404; core services were Running/Automatic.
- Production Mongo remained at 20 accounts, zero enabled federation accounts, zero identities, with `federation_actor_lookup` installed.
- Durable runtime evidence: `docs/test-reports/2026-07-28-christopherbell-dev-activitypub-discovery-foundation-test-report.md` committed at Builder SHA `74f7f83`.

### Current State

- PR #1316 is merged and automatically deployed.
- Production is healthy and intentionally federation-disabled.
- The spoke worktree is clean. GitHub removed the remote feature branch during merge; the local worktree branch remains only as local history.
- Builder plan status is complete; the program spec records discovery foundation completion and the next gate.
- No local candidate process, isolated test database, or acceptance logs remain.

### Follow-ups

The next approved federation gate is signed outbound delivery to a controlled peer. It must remain off in production while implementing strict SSRF/redirect defense, bounded remote actor/key fetching, HTTP signatures, retry/idempotency, payload/time limits, kill switches, and controlled-peer evidence. Only after that gate is safe should opted-in outbound production activation be considered. Inbound follows, then signed/idempotent keep-alives/replies, remain later gates.
