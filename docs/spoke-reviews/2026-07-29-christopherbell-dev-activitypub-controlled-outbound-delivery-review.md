# ChristopherBell.dev ActivityPub Controlled Outbound Delivery Spoke Review

## Document Status

complete

## Related Artifacts

- Program: [Void Public Growth Program](../specs/2026-07-28-void-public-growth-program.md)
- Plan: [Controlled Outbound Delivery Plan](../implementation-plans/2026-07-28-christopherbell-dev-activitypub-controlled-outbound-delivery.md)
- Test report: [Controlled Outbound Delivery Test Report](../test-reports/2026-07-29-christopherbell-dev-activitypub-controlled-outbound-delivery-test-report.md)
- PR: [azurras/christopherbell.dev#1317](https://github.com/azurras/christopherbell.dev/pull/1317)

## Findings

No blockers or warnings.

## Reviewed Spoke

- Repository: `azurras/christopherbell.dev`
- Worktree: `A:\Projects\christopherbell.dev-worktrees\activitypub-outbound-delivery`
- Branch: `codex/activitypub-outbound-delivery`
- Branch commits: `f5c99be0`, `7eb01b9f`, `eab9a9ea`, `2ea0ddb8`
- Merged main SHA: `6c1501070ff518bc040583c4576c2df201dcd3ed`

## Scope Reviewed

Reviewed the fail-closed federation properties, controlled peer allowlist, DNS/IP safety and pinned network transport, exact-byte RSA-SHA256 request signing, canonical Create activity generation, creation-time post eligibility, durable Mongo job/cursor state, leased claims and retries, migration 007 indexes, conditional Spring wiring, production defaults, documentation, and tests.

## Before-Edit Brief Conformance

- Behavior: a newly opted-in eligible post reached one controlled peer and retried 503 to 202 with a stable activity ID.
- Invariants: historical/null posts remain ineligible; production requires no secret or peer while disabled; redirects, proxies, unsafe addresses, and production loopback remain unavailable.
- Boundary: remote effects are conditional on outbound enablement and accept only operator-configured peers after fresh DNS validation.
- Effects/failures: exact bytes are digested/signed, response bodies are discarded, retries are durable and bounded, and stored job state contains no payload, response body, signature, or key.
- Evidence: full check, enabled/disabled runtime, real local Mongo transitions/indexes, green multi-platform CI/CodeQL, automatic deployment, and production no-effect verification all passed.

## House-Style Compliance

The change follows the Jane Street-style coding standard: constrained types at the boundary, explicit fail-closed invariants, deterministic IDs and ordering, localized effects, bounded failure categories, tests at the policy/proxy/configuration/transport/coordinator boundaries, and comments explaining non-obvious security choices rather than restating mechanics.

## Validation Checked

- Local `:website:check`: 1,382 tests, zero failures/errors.
- Runtime: signed 503 to 202 retry with identical activity ID/body; durable SUCCEEDED job after two attempts.
- Kill switch: disabled startup with no key/peer, ineligible post, zero job/request.
- CI: Windows, Ubuntu, macOS, dependency review, and all CodeQL analyses passed on PR and main.
- Production: auto-deployed main SHA, root 200 locally/publicly, discovery 404, migration 007 APPLIED, four indexes present, zero eligible posts/jobs, native services Running/Automatic.

## Residual Scope

Production activation, Update/Delete propagation, broader peer fanout/discovery, inbound follows, and signed/idempotent inbound keep-alives/replies are explicitly separate gates, not defects in this reviewed gate.

## Merge Readiness

Ready. PR 1317 was merged and the automatic production deployment passed while federation remained disabled.
