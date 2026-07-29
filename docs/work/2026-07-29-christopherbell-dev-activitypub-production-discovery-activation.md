# ChristopherBell.dev ActivityPub Production Discovery Activation

- Status: closed

## Objective

Activate the already-implemented read-only ActivityPub discovery surface in production without enabling inbound mutations or uncontrolled outbound delivery. Automate creation and preservation of the required encryption secret, expose deployment preflight evidence, and retain a fail-closed outbound gate until an operator-controlled peer inbox is configured.

## Owner and Context

- Coordinator: Codex in the Builder hub
- User authorization: continue autonomously; prioritize working software and security
- Date opened: 2026-07-29

## Spoke Repository

- Repository: `azurras/christopherbell.dev`
- Authoritative checkout is dirty and behind; preserve it unchanged.
- Implementation will use an isolated worktree from refreshed `origin/main`.

## Related Work

- [Void Public Growth Program](../specs/2026-07-28-void-public-growth-program.md)
- [ActivityPub Discovery Foundation Plan](../implementation-plans/2026-07-28-christopherbell-dev-activitypub-discovery-foundation.md)
- [ActivityPub Controlled Outbound Delivery Plan](../implementation-plans/2026-07-28-christopherbell-dev-activitypub-controlled-outbound-delivery.md)

## Current State

Read-only discovery is active in production at merge `8405cd77d0f1743fe33d70cc80b47e37048090a0`. Inbound and outbound remain disabled, peers remain empty, and production contains zero federation delivery jobs, opted-in identities, and outbound-eligible posts. No real operator-controlled peer inbox is configured locally, so outbound activation remains a separate future gate.

## Security Boundaries

- No committed secret material.
- No weakening of protected production ACLs.
- No inbound ActivityPub mutations.
- No outbound delivery without an explicit controlled peer, not-before cutoff, and kill switch.
- No federation of existing posts or accounts without their existing explicit consent rules.

## Validation

- Full `:website:check`: 1,390 Java tests, zero failures/errors, three skipped; JavaScript checks and boot packaging passed.
- Pester 5.9.0 deployment suite: 37 passed, zero failed/skipped.
- Alternate-port `prod,deploy-smoke` restart proved root and NodeInfo 200, foreign WebFinger 404, inbox POST 403, and a stable 32-byte secret.
- PR 1318 passed Windows, Ubuntu, macOS, dependency review, and CodeQL checks.
- Automatic production deployment rotated the listener from PID 16956 to PID 39760 without prompts.
- Canonical/apex roots and NodeInfo return 200 publicly; foreign WebFinger returns 404 and inbox POST returns 403.
- Production services are Running/Automatic and federation state remains empty.

## Completion

- Pull request: [azurras/christopherbell.dev#1318](https://github.com/azurras/christopherbell.dev/pull/1318)
- Test report: [ActivityPub Production Discovery Activation Test Report](../test-reports/2026-07-29-activitypub-production-discovery-activation.md)
- Spoke review: [ActivityPub Production Discovery Activation Review](../spoke-reviews/2026-07-29-christopherbell-dev-activitypub-production-discovery-activation-review.md)
- Closure: [ActivityPub Production Discovery Activation Closure](../work-closures/2026-07-29-christopherbell-dev-activitypub-production-discovery-activation.md)
