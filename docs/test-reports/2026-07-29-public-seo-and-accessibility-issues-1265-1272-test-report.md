# Public SEO and Accessibility Issues 1265-1272 Test Report

## Document Status

complete

## Story/Issue

GitHub issues `azurras/christopherbell.dev` #1265 through #1272.

## Branch

`codex/issues-1265-1272-20260729`, based on `5de2a8b02941ff7e95b6f2648b7bada9397f68b9`; implementation commits `78ae6dce`, `7bafbe5c`, `7860338c`, `b51b85de`, and `5185e796`.

## App / Environment

Spring Boot website, profile `local`, alternate base URL `http://127.0.0.1:8094`, and disposable Mongo database `cbell_issue_1265_1272_final_20260729`. Both Mongo URI and database were explicitly set; mail was disabled. Production port 8080 and its database were not changed.

## Local Run Details

The final JAR ran hidden as PID `54888` with one active account, one post, and one restaurant in the disposable database. A hand-authored string address initially caused a fixture conversion error; replacing it with the domain's nested `Address` document removed the error without an application change. After testing, the exact PID was stopped, port 8094 was free, and the database was dropped and confirmed absent.

## Test Cases

| Case | Expected | Result |
|---|---|---|
| Full gate | Java, JavaScript, packaging, sensor, policy pass | Pass |
| Private/auth shells | `noindex,nofollow` | Pass |
| Unknown HTML | 404, noindex, no canonical | Pass |
| Protected namespaces | Normal and encoded API denied as JSON | Pass |
| Dynamic pages | Specific titles/canonicals; missing entities 404 | Pass |
| WFL | Correct main/top-rated/favorites indexing policy | Pass |
| Sitemap | Bounded root contains seeded entities; impossible page 404 | Pass |
| The Bell | Doctype, one main/h1, safe blank links | Pass |
| Auth forms | POST fallback, names, autocomplete | Pass |
| Cleanup | Process, listener, database removed | Pass |

## Data Sent

- Seeded `seo_user`, `seo-post-1`, and `seo-restaurant-1` in the disposable database.
- Sent `GET /`, `GET /actuator/health/liveness`, `GET /sitemap.xml`, `GET /sitemap-999999.xml`, `GET /unknown/runtime/path`, `GET /api/admin/secret`, and raw `GET /%61pi/admin/secret`.
- Sent GET requests for login/signup, Void signup, notifications, Back Office, Command Center, WFL pages, The Bell pages, active/missing profile and post pages, and active/missing restaurant pages.
- Inspected rendered canonical/robots tags, form attributes, landmarks, heading count, doctype, and new-tab links.

## Response Received

Status code: 200 from `/` and liveness. Status code: 200 from `/sitemap.xml`; its response body contained the seeded account, post, and restaurant URLs. The impossible sitemap page returned status code: 404. The unknown UI returned status code: 404 with noindex and no canonical. Normal and encoded admin API requests returned status code: 403 and the normal response was JSON. Auth/private shells returned status code: 200 with noindex. Public dynamic pages returned status code: 200 with specific canonicals; missing entities returned status code: 404/noindex. The Bell UI had a doctype, one `main`, one `h1`, and zero unsafe blank links. Login/signup rendered POST forms with username, current-password, and new-password autocomplete semantics.

## Pass / Fail

All accepted runtime cases passed. Final `:website:check` completed 1,418 Java tests with 0 failures/errors and 3 skipped, followed by successful JavaScript, boot JAR, sensor runtime, and policy checks. A prior attempt collided with a still-running Gradle process left by a short command timeout; after stopping the isolated daemon and cleaning only test outputs, the definitive single-process check passed in 2m25s.

## Evidence

- Java XML: `website/build/test-results/test/`.
- JavaScript results: `website/build/test-results/jsTest/`.
- Runtime log excerpt source: `website/build/batch2-runtime-final.out.log` in the isolated worktree.
- Cleanup: `Port8094Listening=false`, database drop `{ ok: 1 }`, database existence `false`.
- Independent review: no remaining Critical or Important blocker; normalized-path regression cases and `git diff --check` passed.

## Bugs / Follow-ups

No unresolved Batch 2 defect remains. Local-profile readiness reports out-of-service on this host even while liveness, root, Mongo-backed sitemap, and exercised pages are healthy; production readiness will be verified after merge.
