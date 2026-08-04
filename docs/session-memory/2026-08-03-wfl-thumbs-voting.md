# 2026-08-03 WFL Thumbs Voting

## 19:19 - Completed WFL thumbs voting delivery

### Request

Replace What's For Lunch 1-5 restaurant ratings with thumbs up/down, converting 3-5 to up and 1-2 to down. Make higher-rated/approved restaurants appear more often in three-restaurant selection, lower-approved restaurants less often, apply Void styling to the WFL page and restaurant profiles, and keep restaurant profiles indexable.

### Project Context

Work was coordinated from `C:\Users\Christopher\Developer\builder` into `azurras/christopherbell.dev`. The authoritative checkout at `A:\Projects\christopherbell.dev` was dirty and preserved. Implementation used the isolated worktree `A:\Projects\christopherbell.dev-worktrees\wfl-thumbs-voting` from refreshed `origin/main` `363bb986581c4d20df3434154844807ce88701e4`, with private Gradle home `A:\Projects\christopherbell.dev-gradle-homes\wfl-thumbs-voting`.

### Work Completed

- Added immutable Mongo migration V013 with a complete preflight pass before any conversion writes.
- Replaced rating domain/models/repositories/services/controllers/session behavior with `UP`/`DOWN` votes and strict legacy-field rejection.
- Added vote aggregates, Top Liked ranking, approval weighting, vote summaries, SSR/JSON-LD, redirects, sitemap updates, thumb controls, race-safe list/profile mutations, Void styling, and ownership documentation.
- Updated Windows production deployment to restore the live backup into a bounded candidate database, validate V013 there, clean it, stop the old writer, and remain forward-only after live migration starts.
- Reviewed the complete branch and resolved all findings.
- Published PR #1349, passed all required CI/security checks, squash-merged as `3b9ee44ba29627c3595b8aebc16612cc2065a885`, and deployed that exact release.

### Decisions

- Convert 3-5 to `UP`, 1-2 to `DOWN`; reject numeric clients immediately.
- Keep `whatsforlunch_ratings` and the unique restaurant/account index.
- Use raw approval then vote count then restaurant ID for Top Liked.
- Use adjusted approval `(up + 1.5) / (up + down + 3)` and selection weights 0.35 at zero approval, 1.0 neutral, and 2.0 at full approval.
- Show `NN% liked · U up · D down`; show `No votes yet` at zero.
- Preserve canonical/indexable restaurant profiles and encode approval JSON-LD on a 0-100 scale.
- Use an approved forward-only outage boundary; never restart the old binary after live V013 begins.

### Validation

- Final Pester production subset: 92/92.
- Final Gradle `:website:check`: `BUILD SUCCESSFUL` in 3m58s; 1,656 Java tests and 336 JavaScript tests passed.
- Alternate candidate on 8094: invalid legacy data failed before writes; valid fixture converted 6 up/3 down and retained indexes; strict API, redirect, sitemap, SEO, weighting sample, authenticated desktop, mobile, persistence, and console checks passed.
- GitHub: Ubuntu, macOS, Windows, Dependency Review, and all CodeQL analyzers passed.
- Production: release junction `3b9ee44b`; 8080 rotated PID 59036 to 74080; readiness/liveness 200; all services running.
- Live Mongo: V013 applied with exact checksum; 87 binary votes, 53 up, 34 down, zero legacy/invalid documents, unique index retained.
- Public/browser: Top Liked, redirect, sitemap, voted/zero/missing profiles, Void WFL/profile rendering, canonical/noindex/JSON-LD, horizontal fit, and empty warning/error console verified.

### Current State

- Builder main already contains the approved spec, plan, work ledger, and local test report checkpoint at `ab2cd78`; this closure update will add completed statuses, closure, and session memory.
- Spoke PR #1349 is merged; the remote feature branch was deleted by the merge command. The local isolated worktree and `.superpowers/sdd/2026-08-03-wfl-thumbs-voting` evidence remain for audit.
- Production is serving the merged release and the disposable candidate databases were removed. The production backup and previous release remain under normal protected retention.

### Follow-ups

No required follow-up remains. Do not edit V013 or its checksum; append a new migration for future vote schema changes. Preserve normal backup/release retention and remove the isolated worktree only during an intentional cleanup pass.
