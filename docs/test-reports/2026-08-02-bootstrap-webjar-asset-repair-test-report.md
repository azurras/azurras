# Bootstrap WebJar Asset Repair Test Report

## Document Status

Complete

## Story/Issue

azurras/christopherbell.dev#1339, Restore Bootstrap assets after WebJar version bump.

## Branch

- codex/issue-1339-bootstrap-assets from origin/main 2b40bd860d9e4e05aa18b4dd63e13a390d41208e.
- Worktree: A:\Projects\christopherbell.dev-worktrees\bootstrap-assets-1339.
- Runtime artifact: website\build\libs\website.jar built from the issue diff.

## App / Environment

- Spring Boot 4.1.0, Java 25.0.3, local profile.
- Base URL: http://localhost:8091 on alternate port 8091.
- Local MongoDB: localhost:27017, database christopherbell.
- Inputs: SPRING_PROFILES_ACTIVE=local, SERVER_PORT=8091,
  APP_PUBLIC_BASE_URL=http://localhost:8091.
- Private Gradle cache: A:\Temp\christopherbell-bootstrap-1339-gradle.
- Production port 8080 remained live on PID 33024 and was never stopped.

## Local Run Details

Exact start command:

    $env:SPRING_PROFILES_ACTIVE='local'
    $env:SERVER_PORT='8091'
    $env:APP_PUBLIC_BASE_URL='http://localhost:8091'
    java -jar website\build\libs\website.jar

- Runtime PID 31020 started Tomcat on 8091 in 5.657 seconds.
- GET /actuator/health/readiness returned HTTP 200 and {"status":"UP"}.
- Live logs were captured in the Codex task transcript.
- PID 31020 was verified as java.exe running the packaged website JAR and
  stopped after testing. Port 8091 had no remaining listener afterward.
- Production port 8080 still listened on PID 33024.

## Test Cases

| Case | Input | Response | Result |
|---|---|---|---|
| Home | GET http://localhost:8091/ | HTTP 200, text/html; charset=UTF-8, 3,981 bytes | PASS |
| Login | GET http://localhost:8091/login | HTTP 200, text/html; charset=UTF-8, 3,802 bytes; 5.3.8 bundle URL | PASS |
| Signup | GET http://localhost:8091/signup | HTTP 200, text/html; charset=UTF-8, 5,832 bytes; 5.3.8 bundle URL | PASS |
| Bootstrap CSS | GET http://localhost:8091/webjars/bootstrap/5.3.8/css/bootstrap.min.css | HTTP 200, text/css, 232,111 bytes; --bs-blue signature | PASS |
| Bootstrap JS | GET http://localhost:8091/webjars/bootstrap/5.3.8/js/bootstrap.bundle.min.js | HTTP 200, text/javascript, 80,496 bytes; Bootstrap v5.3.8 signature | PASS |
| Old CSS | GET http://localhost:8091/webjars/bootstrap/5.3.3/css/bootstrap.min.css | HTTP 403, application/json, 129 bytes | PASS |
| Old JS | GET http://localhost:8091/webjars/bootstrap/5.3.3/js/bootstrap.bundle.min.js | HTTP 403, application/json, 134 bytes | PASS |
| Browser home | Navigate to http://localhost:8091/ | CB \| Home; --bs-blue #0d6efd; margin 0px; border-box; main.css loaded | PASS |
| Browser login | Navigate to http://localhost:8091/login | Login; --bs-blue #0d6efd; margin 0px; 5.3.8 script URL; no warnings/errors | PASS |

## Data Sent

Every runtime input was an anonymous HTTP GET with no body, credentials, cookies,
query parameters, or mutation. Exact asset request paths:

- /webjars/bootstrap/5.3.8/css/bootstrap.min.css
- /webjars/bootstrap/5.3.8/js/bootstrap.bundle.min.js
- /webjars/bootstrap/5.3.3/css/bootstrap.min.css
- /webjars/bootstrap/5.3.3/js/bootstrap.bundle.min.js

The UI input was browser navigation to the home and login URLs.

## Response Received

HTTP response evidence from the running app: home, login, and signup returned
HTTP 200; Bootstrap 5.3.8 CSS returned HTTP 200 text/css with 232,111 bytes;
Bootstrap 5.3.8 JavaScript returned HTTP 200 text/javascript with 80,496 bytes;
obsolete 5.3.3 assets returned HTTP 403 JSON and no asset signature.

UI result evidence from the running app: the browser reported Bootstrap
--bs-blue #0d6efd, zero body margin, and border-box sizing on the home page.
The login page listed /webjars/bootstrap/5.3.8/js/bootstrap.bundle.min.js and
the browser log contained no warning or error entries.

## Pass / Fail

PASS. All issue #1339 local runtime and UI cases passed on alternate port 8091.
The production listener was not modified.

## Evidence

- RED: targeted Node Bootstrap test failed with 5.3.3 actual versus 5.3.8 expected.
- GREEN: the same test passed 2/2.
- Focused Java: SecurityConfigTest passed 16/16; StaticAssetRequestMatcherTest
  passed its GET/POST and current/obsolete/unrelated WebJar boundary assertions.
- JavaScript: :website:jsTest passed 312/312.
- Full: :website:check was BUILD SUCCESSFUL in 3m 20s with 1,610 Java tests,
  0 failures, 0 errors, 3 skipped, and 150 Pester tests passed.
- Runtime evidence includes exact URL, request, status, content type, response
  length, body signature, UI computed style, script URL, PID, and listener state.

## Bugs / Follow-ups

- The local profile later logged an unrelated startup catch-up OpenStreetMap
  import duplicate-key error for normalizedName "aama's kitchen". The app stayed
  healthy and all Bootstrap checks passed. This existing WFL data condition is
  outside issue #1339.
- Production verification remains required after merge and automatic deployment.
