# ChristopherBell.dev ActivityPub Production Discovery Activation Closure

## Final Status

closed

## Related Artifacts

- Work: [Production Discovery Activation](../work/2026-07-29-christopherbell-dev-activitypub-production-discovery-activation.md)
- Program: [Void Public Growth Program](../specs/2026-07-28-void-public-growth-program.md)
- Plan: [Production Discovery Activation Plan](../implementation-plans/2026-07-29-christopherbell-dev-activitypub-production-discovery-activation-plan.md)
- Test report: [Production Discovery Activation Test Report](../test-reports/2026-07-29-activitypub-production-discovery-activation.md)
- Review: [Production Discovery Activation Review](../spoke-reviews/2026-07-29-christopherbell-dev-activitypub-production-discovery-activation-review.md)
- PR: [azurras/christopherbell.dev#1318](https://github.com/azurras/christopherbell.dev/pull/1318)

## Completed Scope

Production now enables the read-only ActivityPub discovery surface. A pre-binding initializer atomically creates or reuses one protected 32-byte encryption secret and refuses alternate production paths, symbolic/non-regular files, malformed lengths, missing parents, and inaccessible storage. An explicit environment secret remains the highest-priority operator override. The `deploy-smoke` profile alone permits an alternate absolute path for isolated testing.

Both NodeInfo routes are mandatory candidate and public deployment checks. Inbound and outbound remain disabled, the peer list remains empty, and old accounts/posts are not enrolled or backfilled.

## Spoke Changes

Repository `azurras/christopherbell.dev` changed in branch commit `1244dd31`. PR 1318 rebased into main at `8405cd77d0f1743fe33d70cc80b47e37048090a0`, and the existing hidden automatic deployment path rolled it into production without interaction.

## Validation

- Full local check: 202 suites, 1,390 tests, zero failures/errors, three skipped; JavaScript task, boot JAR, and sensor verification passed.
- Deployment tests: 37 passed, zero failed/skipped.
- Alternate-port production-profile restart proved root/NodeInfo success, foreign WebFinger and inbox denial, a stable 32-byte key, zero outbound state, and clean shutdown/database cleanup.
- PR checks passed on Windows, Ubuntu, macOS, dependency review, and all CodeQL analyzers.
- Production listener rotated from PID 16956 to 39760. Canonical/apex roots and NodeInfo return 200; foreign WebFinger returns 404; inbox POST returns 403.
- Production has zero delivery jobs, opted-in accounts, identities, outbound-eligible posts, and scan state. MongoDB, website, media worker, and cloudflared are Running/Automatic.
- Protected production files correctly remain unreadable to the non-elevated shell; no ACL was weakened.

## Decisions

- Activated discovery only because no real operator-controlled remote peer is available locally.
- Pinned normal production to the protected ProgramData key path and limited alternate paths to the explicit smoke profile.
- Preserved the stable key across restarts and recorded only a one-way test fingerprint, never its value.
- Used automatic push-to-main deployment and did not request elevation or modify production service state manually.

## Known Gaps and Follow-ups

No defect remains in this activation gate. NodeInfo reports software version `unknown`; this cosmetic release-metadata gap can be fixed separately. Outbound production delivery requires a real controlled peer and its own interoperability evidence. Inbound ActivityPub remains a later security and product gate.

## Resume Point

Future federation work should start from fresh `origin/main` after `8405cd77d0f1743fe33d70cc80b47e37048090a0` in a new isolated worktree. Preserve the production key and keep inbound/outbound disabled until a later gate is designed and verified.
