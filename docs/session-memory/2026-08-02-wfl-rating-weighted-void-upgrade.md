# 2026-08-02 - WFL rating-weighted Void upgrade

## 20:17 - WFL rating-weighted Void upgrade

### Request

The user asked for the three What's for Lunch restaurants to account for ratings so highly rated restaurants appear more often and low-rated restaurants appear less often, and requested that the page CSS be upgraded to the Void style. The user approved the Decision Console direction and autonomous execution through delivery.

### Project Context

Builder coordinated `azurras/christopherbell.dev`. The authoritative checkout `A:\Projects\christopherbell.dev` was dirty and preserved; implementation used isolated worktree `A:\Projects\christopherbell.dev-worktrees\wfl-rating-weighted-void` on `codex/wfl-rating-weighted-void`. The Windows development host is also production, so candidate testing used port 8081 and production port 8080 was not touched until the scheduled deployment pipeline rotated it. Protected ProgramData ACLs were not weakened.

### Work Completed

- Added a rating-weighted selector with a three-rating neutral prior, approved piecewise-linear weights, positive eligibility probability, deterministic injection seam, validation, and sampling without replacement.
- Added one batch MongoDB rating-summary aggregation and wired coordinate/ZIP, daily refresh, and deleted-pick replacement flows through it while preserving persisted order.
- Rebuilt `/wfl` as a dedicated scoped Void Decision Console with three equal desktop cards, a one-column mobile layout, exact rating-influence disclosure, no numeric ranking, accessible focus, and reduced-motion handling.
- Added Java and JavaScript regressions and updated WFL/CSS documentation.
- Independent review found an authenticated rating-color cascade. A failing regression reproduced it; commit `58019300` directly styles both anonymous and authenticated rating paragraphs. Re-review returned ready to merge with no findings.
- PR #1344 passed all required checks, merged as `9c69623049829394f245515b8d1751c9f7579271`, and auto-deployed.

### Decisions

- Adjusted rating is `(ratingSum + 9) / (ratingCount + 3)`; unrated is neutral 3-star.
- Weight anchors are 1=0.35, 2=0.60, 3=1.00, 4=1.50, and 5=2.00 with linear interpolation.
- Selection is without replacement and all eligible candidates retain a positive chance.
- Existing daily and shared-session picks remain stable until normal refresh/reset.
- The new visual layer belongs only to `/wfl`; Top Rated, Favorites, and profile pages retain existing ownership.

### Validation

- Focused final coverage: 8 selector tests, 3 rating query tests, 60 service tests, and 313 JavaScript tests passed.
- Final `:website:check` passed after review correction in 2 minutes 56 seconds, including 150 Pester tests and deployment verification tasks.
- Alternate-port Spring Boot runtime on 8081 passed health, page, today, coordinate, and ZIP HTTP checks plus desktop/mobile Chrome inspection.
- GitHub Windows/macOS/Ubuntu builds, dependency review, and CodeQL for Actions, Java/Kotlin, and JavaScript/TypeScript all passed.
- The merged tree exactly matched the verified feature tree. Production listener rotated from PID 57904 to 55848 and published fingerprinted asset `db0009f03ea001ffc654`.
- Public HTTPS liveness/readiness were `UP`; `/wfl` and three selection paths returned HTTP 200 with real locations and no synthetic metro placeholder.
- Authenticated production browser inspection verified equal cards, zero overflow, corrected 9.06:1 rating contrast, no ranking badges, and no console warnings/errors.

### Current State and Follow-ups

Production is healthy on the merged release. Builder artifacts are closed, and no required follow-up remains. Preserve the isolated worktree until any post-merge operational window ends; its only dirty file is the known baseline `gradlew.bat` line-ending anomaly, which was never staged or committed.
