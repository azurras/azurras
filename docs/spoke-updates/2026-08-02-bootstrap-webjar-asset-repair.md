## Source

- Repository: `azurras/christopherbell.dev`
- Worktree: `A:\Projects\christopherbell.dev-worktrees\bootstrap-assets-1339`
- Branch: `codex/issue-1339-bootstrap-assets`
- Base: `2b40bd860d9e4e05aa18b4dd63e13a390d41208e`
- Final branch head: `facfa97cdb33dd144fbe4aedae5cbc2e45fc2ea3`
- PR: [#1340](https://github.com/azurras/christopherbell.dev/pull/1340)
- Merge: `5bd14e994a6130a32166602a6f272581abc53525`
- Related work: [Bootstrap Asset Regression](../work/2026-08-02-christopherbell-dev-bootstrap-asset-regression.md)

## Status

Complete and deployed.

## Changes

Aligned all Bootstrap WebJar references and both exact security boundaries with
packaged version 5.3.8. Added dependency-derived resource scanning, public
allowlist assertions, and direct `StaticAssetRequestMatcher` GET/POST and
version-boundary tests. Updated pinned-version documentation.

## Validation

- `:website:check`: BUILD SUCCESSFUL; 1,610 Java, 312 JavaScript, and 150 Pester tests.
- Alternate-port packaged app on 8091: readiness UP; current CSS/JS 200; old paths 403; browser rendering passed.
- Independent review: initial matcher-test gap fixed; re-review had no actionable findings and approved merge.
- GitHub: all OS builds, CodeQL lanes, and dependency review passed.
- Production: PID 33024 rotated to 2956; readiness/liveness UP; exact assets and browser rendering passed; required services Running/Automatic.

## Risks and Follow-ups

No issue-scoped gaps remain. The unrelated local-profile WFL duplicate-key
catch-up log remains outside issue #1339.
