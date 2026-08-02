# christopherbell.dev Bootstrap Asset Regression

- Status: active
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

- Project spec: [Restore Bootstrap Assets After WebJar Version Bump](../specs/2026-08-02-restore-bootstrap-assets-after-webjar-version-bump.md) (`ready-for-execution`).
- Implementation plan: pending.
- Source issue: [azurras/christopherbell.dev#1339](https://github.com/azurras/christopherbell.dev/issues/1339).

## Spoke Repositories

- `christopherbell-dev`: authoritative checkout `A:\Projects\christopherbell.dev`;
  dirty, ahead 3, and behind current `origin/main`, so it will remain untouched.
- Planned isolated worktree: `A:\Projects\christopherbell.dev-worktrees\bootstrap-assets-1339`
  from refreshed `origin/main` commit `2b40bd860d9e4e05aa18b4dd63e13a390d41208e`.

## Dispatched Tasks

No sub-agents or external tasks were dispatched.

## Current State

- Production release-versioned custom CSS and JavaScript return HTTP 200.
- `/webjars/bootstrap/5.3.3/css/bootstrap.min.css` and
  `/webjars/bootstrap/5.3.3/js/bootstrap.bundle.min.js` return HTTP 404.
- The packaged `5.3.8` asset paths return HTTP 403 because security matchers
  still allow only the obsolete `5.3.3` namespace.
- Root cause traces to dependency update commit `20290b2f`, which changed only
  `website/build.gradle.kts` from Bootstrap 5.3.3 to 5.3.8.

## Blockers

None.

## Validation

- Reproduced mixed styling in the in-app browser on the public homepage.
- Enumerated public routes and their Bootstrap/script references.
- Confirmed exact 200, 404, and 403 asset responses from production.
- Refreshed `origin/main` and compared dependency, templates, stylesheet
  imports, security matchers, and tests.

## Next Steps

1. Save and review the project spec.
2. Save, validate, and review a literal-line implementation plan.
3. Implement with regression tests in the isolated worktree.
4. Run automated and alternate-port runtime verification.
5. Publish, merge, verify production, close issue #1339, and close this work record.
