# christopherbell.dev Bootstrap Asset Regression

- Status: closed
- Owner/Agent: Codex primary agent
- Started: 2026-08-02

## Objective

Restore complete Bootstrap styling and behavior on production after the packaged
WebJar moved from 5.3.3 to 5.3.8 without matching asset-path and security updates.

## Scope

- Align application-served Bootstrap CSS and JavaScript references with the
  packaged WebJar version.
- Align anonymous static-resource security matchers with the referenced version.
- Add regression coverage that prevents dependency/reference/allowlist drift.
- Validate locally on a non-production port, deliver through PR/CI, and verify
  production before closing the issue.
- Preserve the dirty authoritative checkout at `A:\Projects\christopherbell.dev`
  and implement from refreshed `origin/main` in an isolated worktree.

## Related Specs and Plans

- Project spec: [Restore Bootstrap Assets After WebJar Version Bump](../specs/2026-08-02-restore-bootstrap-assets-after-webjar-version-bump.md) (`complete`).
- Implementation plan: [Implement Bootstrap WebJar Asset Repair](../implementation-plans/2026-08-02-implement-bootstrap-webjar-asset-repair.md) (`complete`).
- Test report: [Bootstrap WebJar Asset Repair Test Report](../test-reports/2026-08-02-bootstrap-webjar-asset-repair-test-report.md) (`complete`).
- Source issue: [azurras/christopherbell.dev#1339](https://github.com/azurras/christopherbell.dev/issues/1339).

## Spoke Repositories

- `christopherbell-dev`: authoritative checkout `A:\Projects\christopherbell.dev`;
  dirty, ahead 3, and behind current `origin/main`, so it will remain untouched.
- Isolated worktree: `A:\Projects\christopherbell.dev-worktrees\bootstrap-assets-1339`
  from refreshed `origin/main` commit `2b40bd860d9e4e05aa18b4dd63e13a390d41208e`.

## Dispatched Tasks

No sub-agents or external tasks were dispatched.

## Current State

- PR [#1340](https://github.com/azurras/christopherbell.dev/pull/1340)
  squash-merged as `5bd14e994a6130a32166602a6f272581abc53525` after all CI,
  CodeQL, and dependency-review checks passed.
- Issue [#1339](https://github.com/azurras/christopherbell.dev/issues/1339)
  is closed by the merged PR.
- Production rotated from PID 33024 to PID 2956. Readiness and liveness are UP;
  Bootstrap 5.3.8 CSS and JavaScript return exact HTTP 200 responses; obsolete
  5.3.3 paths remain HTTP 403.
- Public browser checks confirm Bootstrap computed styles, the 5.3.8 bundle,
  and no console errors. ChristopherBellDev, MongoDB, and cloudflared are
  Running and Automatic.

## Blockers

None.

## Validation

- `:website:check` passed with 1,610 Java tests, 312 JavaScript tests, and 150
  Pester tests; focused security tests passed.
- Alternate-port packaged-app HTTP and browser checks passed on port 8091.
- Independent review found one matcher-test gap; the gap was fixed and the
  re-review returned no actionable findings and Ready to merge: Yes.
- Post-merge production HTTP, readiness/liveness, service, and browser checks
  passed against the automatically deployed release.

## Next Steps

None. Resume only if a new Bootstrap dependency or delivery regression is
observed.
