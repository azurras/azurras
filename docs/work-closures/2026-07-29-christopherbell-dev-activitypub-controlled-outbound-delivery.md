# ChristopherBell.dev ActivityPub Controlled Outbound Delivery Closure

## Final Status

closed

## Related Artifacts

- Central program: [Void Public Growth Program](../specs/2026-07-28-void-public-growth-program.md)
- Implementation plan: [Controlled Outbound Delivery Plan](../implementation-plans/2026-07-28-christopherbell-dev-activitypub-controlled-outbound-delivery.md)
- Runtime report: [Controlled Outbound Delivery Test Report](../test-reports/2026-07-29-christopherbell-dev-activitypub-controlled-outbound-delivery-test-report.md)
- Spoke review: [Controlled Outbound Delivery Review](../spoke-reviews/2026-07-29-christopherbell-dev-activitypub-controlled-outbound-delivery-review.md)
- Pull request: [azurras/christopherbell.dev#1317](https://github.com/azurras/christopherbell.dev/pull/1317)

## Completed Scope

Delivered the second guarded ActivityPub rollout gate. Newly created posts are eligible only when outbound is enabled at creation and the active author explicitly enrolled. Historical missing/null eligibility is false. A scheduler reconciles eligible posts into stable per-post/per-peer Mongo jobs, atomically claims work with leases, rechecks current author/post/peer eligibility, and records bounded success/retry/dead/cancel outcomes.

Outbound requests serialize one canonical Create activity, digest and sign the exact bytes with the account-bound encrypted RSA key, re-resolve the configured peer immediately before sending, reject unsafe or mixed DNS results, pin the connection address while preserving Host/SNI/hostname verification, follow no redirects, use no proxy or implicit retry, discard response bodies, and enforce bounded timeouts/backoff.

Production remains fail-closed: discovery, inbound, and outbound default false; the peer list is empty; the loopback exception is false; no secret is required while disabled.

## Spoke Changes

Repository `azurras/christopherbell.dev` changed through branch commits `f5c99be0`, `7eb01b9f`, `eab9a9ea`, and `2ea0ddb8`. PR 1317 rebased into main at `6c1501070ff518bc040583c4576c2df201dcd3ed` and deployed automatically.

## Validation

- Full local check: 201 suites, 1,382 tests, zero failures/errors.
- Enabled local runtime on 8081 and isolated MongoDB: opted-in post, signed/digested 503 then 202 retry, stable activity ID/body, durable SUCCEEDED job after two attempts, migration 007/indexes present.
- Disabled runtime: started without key/peer, second post ineligible, zero job and no peer request.
- PR/main CI: Windows, Ubuntu, macOS, dependency review, CodeQL all passed.
- Production: listener rotated automatically; local/public root 200; WebFinger/NodeInfo 404; migration 007 APPLIED; all four indexes present; eligible posts zero; jobs zero; MongoDB, ChristopherBellDev, and Cloudflared Running/Automatic.
- Test fixture, ports, and isolated database were cleaned up. The authoritative dirty spoke checkout was never used or modified.

## Decisions

- Kept this as one coherent PR and preserved task-level commits via rebase merge because merge commits are disabled.
- Used Mastodon-compatible `(request-target) host date digest` RSA-SHA256 for this controlled-peer gate; RFC 9421 negotiation remains later work.
- Treated stable activity IDs as receiver-side idempotency keys while unique local post/peer jobs prevent duplicate local scheduling.
- Kept production disabled after deployment; safe implementation does not imply activation authority.

## Known Gaps and Follow-ups

No defect remains in this gate. Next, if explicitly chosen, is a separate production activation gate with controlled remote interoperability evidence. Update/Delete delivery, broader fanout/discovery, inbound follows, and signed/idempotent inbound interactions remain later work.

## Resume Point

Future work should begin from fresh `origin/main` after `6c1501070ff518bc040583c4576c2df201dcd3ed`, use a new isolated worktree and plan, and preserve production-disabled defaults until that later gate passes its own security and runtime evidence.
