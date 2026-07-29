# ChristopherBell.dev ActivityPub Production Discovery Activation

## Objective

Activate the already-implemented read-only ActivityPub discovery surface in production without enabling inbound mutations or uncontrolled outbound delivery. Automate creation and preservation of the required encryption secret, expose deployment preflight evidence, and retain a fail-closed outbound gate until an operator-controlled peer inbox is configured.

## Status

in_progress

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

Discovery, inbound, and outbound federation are disabled in production. Discovery and controlled outbound behavior are implemented and tested. No real operator-controlled peer inbox is configured locally, so this gate will activate discovery only and prepare a safe outbound preflight without inventing a destination.

## Security Boundaries

- No committed secret material.
- No weakening of protected production ACLs.
- No inbound ActivityPub mutations.
- No outbound delivery without an explicit controlled peer, not-before cutoff, and kill switch.
- No federation of existing posts or accounts without their existing explicit consent rules.

## Validation

- Focused and full automated checks.
- Alternate-port production-profile startup and public discovery checks.
- Noninteractive deployment through the existing push-to-main path.
- Post-deploy root, WebFinger, NodeInfo, actor visibility, service, listener, migration, and outbound inactivity checks.

## Next Steps

1. Save and validate the activation design and implementation plan.
2. Implement in an isolated spoke worktree with tests.
3. Open one PR, merge after green CI, and verify the automatic production deployment.
4. Save test, review, closure, and session-memory evidence.
