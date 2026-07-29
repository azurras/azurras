# ChristopherBell.dev ActivityPub Discovery Foundation Closure

## Final Status

complete

## Related Work

- Controlling program: [Void Public Growth Program](../specs/2026-07-28-void-public-growth-program.md)
- Implementation plan: [ActivityPub Discovery Foundation](../implementation-plans/2026-07-28-christopherbell-dev-activitypub-discovery-foundation.md)
- Test report: [ActivityPub Discovery Foundation Test Report](../test-reports/2026-07-28-christopherbell-dev-activitypub-discovery-foundation-test-report.md)
- Session memory: [ActivityPub Discovery Foundation Delivery](../session-memory/2026-07-28-christopherbell-dev-activitypub-discovery-foundation-delivery.md)

## Completed Scope

Delivered the first guarded ActivityPub rollout gate. Accounts now have explicit signup/Profile consent, null-safe legacy opt-out behavior, a stable per-account RSA identity, and AES-256-GCM encryption for private PKCS#8 material. The public read-only surface includes WebFinger, NodeInfo 2.1, local Person actors, bounded active-post outboxes, and bounded local followers/following collections.

The release preserves the safety boundary: discovery, inbound, and outbound flags default off; no remote fetch, HTTP request signing, delivery queue, inbox mutation, remote persistence, or lifespan effect exists. Messages, Music, Shared Folder, reports, and administrative data are excluded. Signup shows the choice disabled when enrollment is unavailable, while an existing enrolled account can always opt out.

## Pull Request and Merge

- Pull request: [#1316](https://github.com/azurras/christopherbell.dev/pull/1316)
- Feature commits: `d7c11fa3`, `50cf469a`, `45b3618b`
- Rebase merge on `main`: `6cd9e397e4ec2c3175ae5c31a95f633b7a7c7c95`
- Automatic production deployment: complete; listener rotated from PID 34768 to PID 33352 and new signup markup appeared locally/publicly.

## Validation

- Full local check: 1,353 Java tests, zero failures/errors, three skips; complete JavaScript and packaged/runtime checks passed.
- Isolated enabled runtime: consented signup, WebFinger, NodeInfo, actor, outbox, followers/following, no-store/CORS/nosniff headers, blocked inbox write, stable restart identity, and opt-out/re-enable behavior passed.
- Isolated disabled runtime: the app started without an encryption key, signup disabled enrollment, and public federation endpoints returned 404.
- GitHub: Ubuntu, macOS, Windows, dependency review, and all CodeQL analyses passed on PR #1316.
- Production: home/signup returned 200 locally/publicly; federation choice was present but disabled; NodeInfo/actor returned 404; all core services were Running/Automatic.
- Production data: 20 accounts, zero federation-enabled accounts, zero identities, and `federation_actor_lookup` present.
- Cleanup: port 8081 closed, isolated database dropped, and temporary scripts/logs removed.

## Decisions

- Existing and API-omitted accounts remain opted out; new browser signups default on only when the deployment is actually configured to enroll identities.
- Disabling discovery does not remove stored identity material, so re-enable preserves the actor and public key; it immediately removes public discoverability.
- Actor origin is derived only from the existing canonical browser public base URL.
- Outbox content is read-only, active-only, bounded to 20, cursor-stable, and HTML-escaped.
- One coherent PR carried the rollout gate. Reviews were limited to correctness, interoperability, and security boundaries.
- Production remains default-disabled until protected key installation and explicit activation are separately approved as an operational change.

## Known Gaps and Follow-ups

No defect remains in this gate. This is not full federation.

The next approved gate is signed outbound delivery to a controlled peer, still off in production. It must add bounded remote actor/key discovery, strict SSRF and redirect controls, HTTP signing, retry/idempotency, payload limits, kill switches, and controlled-peer evidence before any outbound production activation. Inbound follows and mutations remain later gates.

Future work should start from merged `main`. Preserve the dirty authoritative checkout at `A:\Projects\christopherbell.dev`; the isolated feature worktree is clean.
