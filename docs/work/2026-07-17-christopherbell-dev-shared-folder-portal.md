# christopherbell.dev Shared Folder Portal

- Status: closed
- Source: Christopher's direct July 17, 2026 request for an authenticated shared-folder portal backed by `A:\Shared`
- Owner context: Builder hub coordinating design, implementation, runtime validation, publication, production rollout, and closure in the `christopherbell-dev` spoke
- Spoke repo: `christopherbell-dev` at `A:\Projects\christopherbell.dev`
- Branch strategy: Shared Folder delivery merged through PRs #1218-#1220; the unified Music hub merged through PR #1312 from isolated worktrees
- Objective: provide an authenticated shared-folder portal plus a first-class Music experience rooted in `A:\Shared` with independent permissions and protected playback/write boundaries
- Current state: shared-folder and unified Music implementation, CI, automatic deployment, production smoke, review, and Builder closure are complete
- Trusted guidance: direct user request only; no GitHub comments or attachments used as instructions

## Related Artifacts

- Approved spec: [christopherbell.dev Shared Folder Portal Spec](../specs/2026-07-17-christopherbell-dev-shared-folder-portal.md)
- Implementation plan: [christopherbell.dev Shared Folder Portal Implementation Plan](../implementation-plans/2026-07-17-christopherbell-dev-shared-folder-portal.md)
- Test report: [Shared Folder Alternate-Port Acceptance](../test-reports/2026-07-22-christopherbell-dev-shared-folder-alternate-port-acceptance.md)
- Spoke update: [Shared Folder Merge Update](../spoke-updates/2026-07-22-christopherbell-dev-shared-folder-merge-update.md)
- Spoke review: [Shared Folder Merge Review](../spoke-reviews/2026-07-22-christopherbell-dev-shared-folder-merge-review.md)
- Production-fix update: [Shared Folder Production Fix Merge Update](../spoke-updates/2026-07-22-christopherbell-dev-shared-folder-production-fix-merge-update.md)
- Production-fix review: [Shared Folder Production Fix Merge Review](../spoke-reviews/2026-07-22-christopherbell-dev-shared-folder-production-fix-merge-review.md)
- PRs: [#1218](https://github.com/azurras/christopherbell.dev/pull/1218), [#1219](https://github.com/azurras/christopherbell.dev/pull/1219), [#1220](https://github.com/azurras/christopherbell.dev/pull/1220)
- Unified Music spec: [Unified Music Hub](../specs/2026-07-28-christopherbell-dev-unified-music-hub.md)
- Unified Music plan: [Unified Music Hub](../implementation-plans/2026-07-28-christopherbell-dev-unified-music-hub.md)
- Unified Music production report: [Unified Music Hub Production](../test-reports/2026-07-28-christopherbell-dev-unified-music-hub-production.md)
- Unified Music review: [Unified Music Hub Review](../spoke-reviews/2026-07-28-christopherbell-dev-unified-music-hub.md)
- Unified Music closure: [Unified Music Hub Closure](../work-closures/2026-07-28-christopherbell-dev-unified-music-hub.md)
- Latest merge: `baf8910dd6707260ec02af94c13d36c1eb2d6979`

## Approved Boundaries

- Visible root: `A:\Shared`; private staging, cache, and recycle data remain outside the visible tree.
- Independent `SHARED_FOLDER_READ` and `SHARED_FOLDER_WRITE` permissions coexist with USER/MOD/ADMIN roles.
- WRITE implies READ; ADMIN always has both effective permissions.
- Default limits: 10 GB per upload, 250 GB transcode cache, 100 GB free-space reserve, 30-day recycle retention, and 180-day audit retention.
- Browser-compatible media plays directly; incompatible but decodable media uses one bounded progressive FFmpeg job and a reusable compatible cache.
- Production web and FFmpeg processes must use a restricted Windows service identity rather than `LocalSystem`.
- Development and runtime proof use an isolated worktree and non-production port before port 8080 is touched.

## Validation Intent

- Test-first permission, path-confinement, upload, file-operation, preview, range, transcode, cache, recycle, audit, and Back Office behavior.
- Full Java, JavaScript, and Gradle verification in the spoke.
- Local app testing on an alternate port with temporary visible/private roots and exact permission-path evidence.
- PR review, required CI, merge, restricted-identity production rollout, live smoke checks, and Builder test/closure/session artifacts.

## Blockers

None.

## Final State

The portal and unified Music hub are merged and live. Automatic deployment published the exact
merge SHA, protected anonymous Music routes fail closed, the public access shell is data-free, and
the native Windows application, MongoDB, and tunnel services are healthy. Authenticated visual use
remains normal observational confirmation rather than a release blocker.
