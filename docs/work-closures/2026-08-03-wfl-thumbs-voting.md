# WFL Thumbs Voting Closure

## Final Status

closed

## Central Work

- [What's For Lunch Thumbs Voting](../work/2026-08-03-wfl-thumbs-voting.md)
- [Specification](../specs/2026-08-03-wfl-thumbs-voting.md)
- [Implementation plan](../implementation-plans/2026-08-03-wfl-thumbs-voting.md)
- [Local runtime test report](../test-reports/2026-08-03-wfl-thumbs-voting-test-report.md)

## Completed Scope

- Replaced 1-5 restaurant ratings with strict `UP` and `DOWN` votes.
- Added V013 full-collection preflight and deterministic 3-5 to up, 1-2 to down conversion while retaining the existing collection and unique restaurant/account index.
- Replaced public/private API, aggregation, Top Liked ordering, session/service, and browser contracts with binary votes and approval summaries.
- Weighted daily, nearby, and shared-session selection by smoothed approval so well-liked restaurants appear more often and disliked restaurants less often while no-vote restaurants remain eligible.
- Added thumb controls to WFL picks, restaurant profiles, Favorites, and Top Liked with idempotent/race-safe state updates.
- Applied Void styling to the WFL experience and restaurant profiles, retained indexable canonical profiles, emitted 0-100 approval JSON-LD, and kept zero-vote/missing-profile crawler behavior correct.
- Hardened Windows deployment to validate V013 against a disposable restored clone before a stop-old-writer, forward-only live migration boundary.

## Spoke Repository and Publication

- Repository: `azurras/christopherbell.dev`
- Isolated worktree: `A:\Projects\christopherbell.dev-worktrees\wfl-thumbs-voting`
- Reviewed branch commit: `fbab5e8816c66fd8c46147a95cf43f0832c3b341`
- Pull request: [#1349 Replace WFL ratings with thumbs voting](https://github.com/azurras/christopherbell.dev/pull/1349)
- Squash merge: `3b9ee44ba29627c3595b8aebc16612cc2065a885`
- Required checks: Ubuntu, macOS, Windows, Dependency Review, CodeQL Actions, CodeQL Java/Kotlin, and CodeQL JavaScript/TypeScript all passed.

## Validation

- Fresh diff whitespace validation passed.
- Windows production common/operations/deploy Pester passed 92/92.
- `:website:check` completed `BUILD SUCCESSFUL`; supporting totals were 1,656 Java tests with zero failures and 336/336 JavaScript tests.
- Packaged alternate-port validation on 8094 passed invalid-data preflight, successful conversion, strict API rejection, ordering, weighting sample, redirect/sitemap/SEO, desktop/mobile UI, idempotence, persistence, and clean console checks.
- Final whole-branch review passed with no critical, important, or minor findings.

## Production

- Protected release: `C:\ProgramData\christopherbell.dev\releases\3b9ee44ba29627c3595b8aebc16612cc2065a885`
- Listener rotation: port 8080 PID 59036 to PID 74080.
- Services: ChristopherBellDev, MongoDB, and cloudflared running; readiness and liveness HTTP 200.
- V013: `APPLIED` with checksum `c10c2769b37044d866224770f7fb8b0877e02c2457c53d33ee25eeb879ab86f7`.
- Live vote shape: 87 total, 53 up, 34 down, zero legacy rating fields, zero invalid votes, unique `restaurant_account_unique` retained.
- Public verification: Top Liked API/page 200; `/wfl/top-rated` 308 to `/wfl/top-liked`; sitemap current; canonical indexable profile and 0-100 JSON-LD correct; zero-vote profile omits aggregate; missing profile is 404/noindex; `/wfl` and profiles render Void styling without horizontal overflow or browser warnings/errors.

## Decisions

- Legacy numeric clients are rejected rather than translated.
- Approval weighting uses `(up + 1.5) / (up + down + 3)` and maps approval to weights from 0.35 through 2.0.
- The live migration is forward-only after the old writer stops. The protected deployment retained the prior release but did not restart it after V013 began.

## Known Gaps and Follow-ups

No required gaps or follow-ups remain. V013 is immutable; future schema changes must use a new migration ID. The isolated worktree and scratch evidence remain available for audit and can be removed later under normal worktree cleanup.
