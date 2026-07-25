# ChristopherBell.dev Public Delivery Acceptance

## Document Status

complete

This report covers the isolated local runtime checkpoint for issues #1122, #1123, #1124, and
#1138. Pull-request, production, and issue-closure checkpoints remain separate.

## Story/Issue

- [#1122](https://github.com/azurras/christopherbell.dev/issues/1122): make both public hostnames
  serve or canonically redirect public routes and expand deployment smoke coverage.
- [#1123](https://github.com/azurras/christopherbell.dev/issues/1123): publish valid `robots.txt`
  and `sitemap.xml` metadata.
- [#1124](https://github.com/azurras/christopherbell.dev/issues/1124): expose bounded liveness and
  readiness probes and require them during deployment.
- [#1138](https://github.com/azurras/christopherbell.dev/issues/1138): add release-scoped static
  asset URLs and safe cache headers.
- Builder plan: `docs/implementation-plans/2026-07-25-public-delivery-issues-1122-1124-1138.md`.

## Branch

- Repository: `azurras/christopherbell.dev`
- Branch: `codex/public-delivery-1122-1124-1138`
- Commit: `c5b189c517f3f1fe44d7f0542595aa9d14c3221b`
- Base: `origin/main` at `1c4a532b`

## App / Environment

- App: ChristopherBell.dev Spring Boot website on native Windows.
- Profile: `local`.
- Candidate URL: `http://127.0.0.1:8091`.
- Candidate release identifier: `runtime-public-delivery`.
- Candidate worktree:
  `A:\Projects\christopherbell.dev-worktrees\public-delivery-1122-1124-1138`.
- Production remained on `http://127.0.0.1:8080` and was not restarted or modified.

## Local Run Details

The candidate started from the isolated worktree with `GIT_COMMIT=runtime-public-delivery`, the
`local` profile, and `--server.port=8091`. The health probes were polled until both were healthy
before the route matrix ran. After evidence capture, the exact process listening on port 8091 was
validated against the worktree command line and stopped. Port 8091 was confirmed closed, and the
production root on port 8080 still returned status code 200.

## Test Cases

| Case | Behavior | Checks | Result |
| --- | --- | ---: | --- |
| Public pages | Home, Blog, What's for Lunch, and Raising Canes pages render anonymously. | 4 | Pass |
| Search metadata | Robots and sitemap return canonical, parseable metadata with correct media types. | 2 | Pass |
| Static delivery | Favicon and rendered release-scoped CSS/JavaScript/module imports resolve. | 5 | Pass |
| Cache policy | Release-scoped assets are immutable for one year; direct assets use one hour. | 4 | Pass |
| Health boundary | Liveness/readiness are public and healthy; aggregate and component health stay private. | 4 | Pass |
| Deployment scripts | Two-host route matrix, configuration validation, release identity, and rollback checks. | 243 | Pass |
| Full repository | Clean Java, JavaScript, packaging, and sensor-runtime verification. | 21 tasks | Pass |

## Data Sent

- Request matrix: `GET /`, `GET /blog`, `GET /wfl`, `GET /canes-box-tracker`,
  `GET /robots.txt`, `GET /sitemap.xml`, and `GET /favicon.ico`.
- Anonymous `GET` requests to `/`, `/blog`, `/wfl`, `/canes-box-tracker`, `/robots.txt`,
  `/sitemap.xml`, and `/favicon.ico`.
- Anonymous `GET` requests to `/actuator/health/liveness`, `/actuator/health/readiness`,
  `/actuator/health`, and `/actuator/health/mongo`.
- Anonymous `GET` requests to release-scoped `/runtime-public-delivery/css/main.css`,
  `/runtime-public-delivery/js/app.js`, and `/runtime-public-delivery/js/components/nav.js`.
- Anonymous `GET` request to direct `/js/app.js` and missing
  `/runtime-public-delivery/js/missing.js` to confirm cache-boundary behavior.
- The Windows deployment Pester suite supplied valid and invalid two-host configurations,
  release identifiers, route results, switch failures, and rollback results through mocks.

## Response Received

- `/`, `/blog`, `/wfl`, and `/canes-box-tracker`: status code 200 with `text/html`.
- `/robots.txt`: status code 200 with `text/plain`, `Cache-Control: no-cache`, wildcard allow,
  and the canonical `https://www.christopherbell.dev/sitemap.xml` reference.
- `/sitemap.xml`: status code 200 with `application/xml`, `Cache-Control: no-cache`, and nine
  canonical `https://www.christopherbell.dev/` locations.
- `/favicon.ico`: status code 200 with `image/x-icon` and one-hour public caching.
- `/actuator/health/liveness` and `/actuator/health/readiness`: status code 200 with body
  `{"status":"UP"}` and no-store headers.
- `/actuator/health` and `/actuator/health/mongo`: status code 403, preserving private health
  detail.
- Rendered home markup emitted `/runtime-public-delivery/css/main.css`,
  `/runtime-public-delivery/js/app.js`, and `/runtime-public-delivery/js/home.js`.
- Release-scoped CSS, app JavaScript, and relative module import: status code 200 with
  `Cache-Control: public, max-age=31536000, immutable`.
- Direct `/js/app.js`: status code 200 with `Cache-Control: public, max-age=3600`.
- Missing release-scoped asset returned an error with no-store headers and did not receive an
  immutable directive.

## Pass / Fail

Pass. The isolated runtime satisfied every acceptance route, metadata, probe, access-control,
asset-resolution, and cache-header assertion. The final post-rebase clean build completed all 21
tasks successfully in 1 minute 47 seconds. The full Windows deployment suite passed 239 of 243
tests with zero failures and four expected environment-only skips.

Test-first evidence was retained during development: the initial public-delivery configuration
test failed four contract groups before implementation, and the deployment suite failed six
public-host/route checks before the new behavior was added. Runtime testing then found and drove
fixes for release-scoped security matchers and servlet filter ordering/header canonicalization.

## Evidence

- `PublicDeliveryConfigurationTest`: 6/6 focused tests passed.
- `gradlew.bat clean build`: `BUILD SUCCESSFUL`, 21 actionable tasks executed.
- `Invoke-Pester -Path .\ops\production\windows\tests`: total 243, passed 239, failed 0,
  skipped 4.
- Runtime route matrix: 11 endpoint checks and 5 asset-boundary checks captured directly from
  port 8091.
- `git diff --check`: no errors.
- Production continuity: port 8080 root status code 200 after candidate shutdown; port 8091
  confirmed closed.

## Bugs / Follow-ups

No local acceptance defect remains for these four issues. The apex-host Cloudflare routing or
canonical redirect and the protected production `publicUrls` configuration must be completed
before deployment, followed by live two-host verification. Pull-request checks, merge,
production rollout, issue closure, and Builder campaign closeout remain.
