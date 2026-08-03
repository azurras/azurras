# What's For Lunch Thumbs Voting

## Status

active

## Objective

Replace the WFL 1–5 restaurant rating system with thumbs-up/thumbs-down voting, migrate existing data deterministically, rename and re-rank the public leaderboard, preserve indexable restaurant profiles, and align weighted restaurant selection with smoothed binary approval.

## Owner and Context

- Hub: `C:\Users\Christopher\Developer\builder`
- Spoke: `A:\Projects\christopherbell.dev`
- Requested by: direct user request on 2026-08-03
- Delivery model: approved design, durable spec and implementation plan, isolated refreshed-origin worktree, regression-first migration and code changes, alternate-port migrated-database/browser validation, PR/CI/merge, protected production deployment, and final Builder closeout

## Approved Decisions

- Convert stored `3–5` ratings to `UP` and `1–2` to `DOWN`.
- Use one clean V013 in-place data migration; do not retain dual-schema reads.
- Reject old numeric writes immediately.
- Show approval percentage plus up/down counts.
- Rename Top 10 Rated to Top 10 Liked and permanently redirect the old public page URL.
- Rank by raw approval percentage, then total votes, then stable restaurant ID.
- Use a three-vote neutral prior and preserve the selection-weight range from `0.35×` through `2.0×`.

## Related Artifacts

- Specification: [What's For Lunch Thumbs Voting](../specs/2026-08-03-wfl-thumbs-voting.md), ready for user review
- Implementation plan: pending written-spec approval
- Test report: pending implementation
- Spoke update/review: pending implementation
- Closure/session memory: pending final delivery

## Spoke Repositories

- `azurras/christopherbell.dev` at `A:\Projects\christopherbell.dev`

## Current State

Design sections are approved. Current `origin/main` is `363bb986581c4d20df3434154844807ce88701e4`. The authoritative spoke checkout is heavily dirty and stale, so implementation will use a new isolated worktree from refreshed `origin/main`. The current rating model, Mongo aggregation, three browser surfaces, profile SSR/JSON-LD, daily selector, shared-session selector, and V013 migration boundary have been inventoried.

## Blockers

None. Written specification review is the next required gate.

## Validation

Read-only exploration confirmed one integer rating per account/restaurant, sum/count public aggregation, numeric API contracts, Top 10 Rated ordering, current confidence-adjusted weighted sampling, server-rendered profile aggregateRating, and the versioned migration framework through V012.

## Next Steps

1. Commit and push the approved written specification and work record.
2. Obtain user review of the written specification.
3. Create, review, validate, commit, and push the implementation plan.
4. Implement and complete the default delivery loop through production and Builder closure.
