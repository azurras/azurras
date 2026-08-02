# Restore Bootstrap Assets After WebJar Version Bump

## Document Status

Complete

## Purpose

Restore the Bootstrap CSS and JavaScript that production pages lost when the
packaged Bootstrap WebJar changed from 5.3.3 to 5.3.8 without corresponding
changes to application asset references and security matchers.

## Background

GitHub issue [#1339](https://github.com/azurras/christopherbell.dev/issues/1339)
tracks a production regression reported on 2026-08-02. The site still serves its
release-versioned custom `main.css` and JavaScript, so some styling and behavior
remain. However, `main.css` imports Bootstrap 5.3.3 and several templates load
the 5.3.3 bundle while the packaged dependency is Bootstrap 5.3.8.

Current production evidence:

- `/webjars/bootstrap/5.3.3/css/bootstrap.min.css` returns HTTP 404.
- `/webjars/bootstrap/5.3.3/js/bootstrap.bundle.min.js` returns HTTP 404.
- Corresponding `5.3.8` resources return HTTP 403 because both anonymous
  security matchers still permit only the obsolete `5.3.3` namespace.
- Release-versioned custom assets return HTTP 200 and immutable cache headers.

The mismatch began in dependency update commit `20290b2f`, which changed the
Bootstrap dependency to 5.3.8 but did not update direct references or allowlists.

## Goals

- Serve the packaged Bootstrap 5.3.8 CSS and JavaScript successfully to anonymous
  visitors.
- Restore full styling and Bootstrap-powered behavior on every affected page.
- Keep Bootstrap self-hosted through the pinned WebJar.
- Prevent a future WebJar update from silently drifting away from references and
  security matchers.
- Deliver the repair through automated tests, alternate-port runtime testing,
  required CI, merge, and production verification.

## Non-Goals

- Redesign site styling or navigation.
- Switch Bootstrap to a CDN.
- Upgrade Bootstrap beyond the already packaged 5.3.8 release.
- Refactor unrelated static-asset versioning or security configuration.
- Modify the dirty authoritative checkout.

## Requirements

### Functional

1. `main.css` must import the packaged Bootstrap 5.3.8 stylesheet.
2. Every server-rendered template that loads the Bootstrap bundle must request
   the packaged 5.3.8 bundle.
3. The development feed harness must reference the packaged 5.3.8 stylesheet.
4. Anonymous `GET` requests to the exact Bootstrap 5.3.8 WebJar namespace must
   pass both static-resource classification and Spring Security authorization.
5. Obsolete Bootstrap 5.3.3 references must not remain in production source,
   templates, security matchers, documentation, or tests.
6. Other WebJar namespaces and non-GET access must remain outside the permitted
   static-resource boundary.

### Non-Functional

1. The dependency remains pinned to `org.webjars:bootstrap:5.3.8`.
2. Content Security Policy continues to allow self-hosted styles/scripts without
   adding an external Bootstrap host.
3. Static WebJar behavior must remain least-privilege and method-specific.
4. Regression tests must make dependency/reference/allowlist drift visible in CI.

## Proposed Approach

Use one repository constant only where Java security configuration benefits from
shared construction; do not add runtime indirection to static HTML or CSS.

- Replace all intentional `5.3.3` Bootstrap WebJar paths with `5.3.8`.
- Update the static-asset matcher and anonymous rule registry together.
- Update security tests to exercise the 5.3.8 CSS and bundle paths and retain
  negative tests for near-miss/method boundaries.
- Add a narrow source-consistency test that reads the Gradle dependency version
  and asserts no stale version is referenced by production resources or security
  configuration. Prefer an existing source-contract test pattern if present.
- Update documentation that names the pinned version.
- Do not touch feature code or custom CSS beyond the import URL.

## Files and Modules Involved

Expected spoke files under `website`:

- `build.gradle.kts` (dependency source of truth; expected test input, no version change)
- `src/main/resources/static/css/main.css`
- `src/main/resources/static/css/README.md`
- `src/main/resources/static/dev/feed-harness.html`
- Bootstrap-loading templates under `src/main/resources/templates/`
- `src/main/java/dev/christopherbell/configuration/security/SecurityConfig.java`
- `src/main/java/dev/christopherbell/configuration/security/StaticAssetRequestMatcher.java`
- `src/main/java/dev/christopherbell/configuration/security/README.md`
- `src/test/java/dev/christopherbell/configuration/SecurityConfigTest.java`
- A narrow consistency test in the configuration/static-asset test area if an
  existing suitable test does not already cover version synchronization.

Implementation occurs in an isolated worktree from refreshed `origin/main`.

## Validation Plan

### Automated

- Start with a failing test that demonstrates 5.3.8 is not currently permitted
  and/or referenced consistently.
- Run focused static-asset/security tests.
- Run the repository JavaScript test task because templates and browser assets
  are affected.
- Run `:website:check` using a private `GRADLE_USER_HOME`.
- Run source searches proving no stale `5.3.3` reference remains in the scoped
  production/test/documentation files.

### Local Runtime

- Build and start the packaged website on a non-8080 port.
- Request the homepage, login page, and signup page.
- Request both Bootstrap 5.3.8 CSS and bundle paths anonymously and capture
  HTTP status, content type, and response length.
- Confirm the same obsolete 5.3.3 paths return 404 or remain unavailable.
- Use the browser against the alternate-port app to confirm Bootstrap-derived
  computed styling is applied and the affected pages have no failed Bootstrap
  requests.

### Delivery and Production

- Commit and push the spoke branch.
- Open a pull request linked to issue #1339.
- Wait for required CI checks and address only in-scope failures.
- Merge after required gates pass.
- Verify automatic production deployment by listener rotation or equivalent
  indirect evidence without weakening protected ACLs.
- Re-request public Bootstrap CSS and JavaScript and confirm HTTP 200.
- Recheck affected public pages in the browser.
- Close issue #1339 only after production verification.

## Acceptance Criteria

- Public Bootstrap 5.3.8 CSS and JavaScript return HTTP 200 with the expected
  content types.
- Homepage and authentication pages render with Bootstrap styles restored.
- No production page references Bootstrap 5.3.3.
- Focused tests, JavaScript tests, and `:website:check` pass.
- Required PR CI passes and the PR is merged.
- Production verification confirms the repair before issue closure.
- Builder test report, work closure, and session memory are persisted.

## Risks and Mitigations

- Risk: changing the URL without the security matcher still yields 403.
  Mitigation: test both security registries and runtime HTTP responses.
- Risk: updating only known templates leaves another stale reference.
  Mitigation: repository-wide scoped search plus consistency coverage.
- Risk: testing on the production listener interrupts the live site.
  Mitigation: use a packaged app on a non-8080 port first.
- Risk: the authoritative checkout contains extensive unrelated work.
  Mitigation: use a refreshed-origin isolated worktree and never clean or reset
  the authoritative checkout.

## Open Questions

None. The packaged version, affected references, security boundary, validation
surface, and delivery path are all confirmed.
