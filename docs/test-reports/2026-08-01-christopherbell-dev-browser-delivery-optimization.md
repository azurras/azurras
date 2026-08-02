# christopherbell.dev Browser Delivery Optimization Test Report

## Document Status

complete

## Story/Issue

Approved repository-wide website performance, scalability, and shared-library optimization campaign; browser delivery phase.

## Branch

- Spoke branch: `codex/browser-delivery-optimization`
- Commit under test: `c530fe3279eaab6ee047a424b7dda6190fd14832`
- Base commit: `c4d60ce0c92281c201d063cfd6a07563f4a7b230`
- Pull request: `azurras/christopherbell.dev#1337`
- Merged commit: `95d805658beaa4c62a8b5e56af9bbf1c0aca66a6`

## App / Environment

- App: `christopherbell.dev` Spring Boot website.
- Candidate URL: `http://127.0.0.1:8093`.
- Profiles: `local,deploy-smoke`.
- Disposable Mongo database: `christopherbell_browser_verify_20260801_c530fe32`.
- Scheduling: `APP_SCHEDULING_ENABLED=false`.
- Mail: `APP_MAIL_ENABLED=false`.
- JWT secret: temporary verification-only value, not retained.
- Production URL/listener: port 8080, PID `39812` during candidate testing;
  automatically rotated to PID `33336` after merge without a manual restart.
- Browser: Codex in-app Chromium browser at desktop and 390 by 844 mobile viewports.

## Local Run Details

The final candidate JAR ran as PID `16160` from
`A:\Projects\christopherbell.dev-worktrees\browser-delivery-optimization`.
The sanitized start form was:

```powershell
$env:SPRING_PROFILES_ACTIVE = 'local,deploy-smoke'
$env:SERVER_PORT = '8093'
$env:SPRING_MONGODB_URI = 'mongodb://127.0.0.1:27017'
$env:SPRING_MONGODB_DATABASE = 'christopherbell_browser_verify_20260801_c530fe32'
$env:APP_SCHEDULING_ENABLED = 'false'
$env:APP_MAIL_ENABLED = 'false'
$env:APP_JWT_SECRET = '<redacted-verification-only-secret>'

Start-Process -WindowStyle Hidden `
  -FilePath 'java.exe' `
  -ArgumentList @('-jar', 'website\build\libs\website.jar') `
  -RedirectStandardOutput '<temporary-stdout-log>' `
  -RedirectStandardError '<temporary-stderr-log>' `
  -PassThru
```

The final app started in 5.768 seconds. After testing, PID 16160 was stopped, port
8093 was confirmed free, the exact disposable database was dropped and verified
absent, and the temporary runtime logs were deleted. Production port 8080
remained PID 39812. MongoDB, ChristopherBellDev, and cloudflared remained
Running/Automatic.

## Test Cases

| Case | Expected | Result |
| --- | --- | --- |
| Candidate health | Root, liveness, and readiness return 200; health groups are UP | Pass |
| Lightweight public routes | Login, signup, VIN, and ZIP pages render without Blog, Gallery, or player runtime modules | Pass |
| Initial JavaScript graph | Raw static graph stays at or below 86,434 bytes | Pass: 64,123 bytes across 10 modules, down from 174,433 bytes across 14 modules |
| Demand-loaded Blog | Blog component and module load only on `/blog` | Pass |
| Demand-loaded Gallery | Gallery component and module load only on `/photos` | Pass |
| Feature stylesheet ownership | Command Center, Shared Folder, and Void pages reference their dedicated versioned stylesheet | Pass |
| Player stylesheet ownership | Player stylesheet is not part of anonymous initial page delivery and loader tests prove one-time insertion before mount | Pass |
| Static content fingerprint | Repeated builds match; backend-only changes do not change the value; static changes do; file boundaries are unambiguous | Pass |
| Asset caching | Versioned assets are immutable for one year; direct paths remain bounded to one hour | Pass |
| Responsive public feature page | `/void/explore` has its dedicated stylesheet and no horizontal overflow at 390 by 844 | Pass |
| Browser console | No warning or error entries during Login, Blog, Photos, and Void checks | Pass |
| Full candidate gate | Browser, Java, packaging, sensor, and deployment-context checks pass from a fresh rerun | Pass |
| Cleanup and production isolation | Candidate and disposable data are removed; port 8080 and services remain unchanged | Pass |
| Pull request and main CI | PR and post-merge Linux, macOS, Windows, CodeQL, and dependency checks are green | Pass |
| Production delivery | Listener rotates automatically; local/public health and static delivery match the merged tree | Pass |

## Data Sent

Read-only HTTP GET requests were sent to:

- `/`, `/actuator/health/liveness`, and `/actuator/health/readiness`;
- `/login`, `/signup`, `/vin-decoder`, and `/zip-coordinates`;
- `/blog` and `/photos`;
- `/command-center`, `/shared`, `/void/explore`, and `/void/topic/test`;
- fingerprinted CSS and JavaScript paths under
  `/520a074536dc2f6e737d/` and the direct `/css/main.css` path.

The browser navigated anonymously to Login, Blog, Photos, Command Center, and
Void Explore. No credentials, form values, uploads, or external-service data were
sent. Command Center correctly redirected the anonymous browser to the site 404
page after its protected API denied access; its data-free shell and dedicated
stylesheet were therefore verified over direct HTTP rather than as an
authenticated visual flow.

## Response Received

- Status code: 200 OK for every canonical page and health endpoint listed below.
- Root, liveness, readiness, Login, Signup, VIN Decoder, ZIP Coordinates, Blog,
  Photos, Command Center, Shared Folder, Void Explore, and Void Topic returned
  HTTP 200 over their canonical routes.
- Liveness and readiness returned `{"status":"UP"}`.
- Rendered local CSS and JavaScript URLs used the exact namespace
  `/520a074536dc2f6e737d/`.
- Browser asset inventory on Login contained the 10-module global graph plus the
  page's login module and Bootstrap; it contained no Blog, Gallery, Music,
  Shared Folder, or site-media-player implementation module.
- `/blog` mounted `blog-posts` and fetched only
  `/js/components/blog.js` as its page feature.
- `/photos` mounted `photo-gallery` and fetched only
  `/js/components/gallery.js` as its page feature.
- `/command-center` linked `/css/command-center.css`, `/shared` linked
  `/css/shared-folder.css`, and Void Explore and Topic linked
  `/css/void-discovery.css`, all through the fingerprinted namespace.
- Fingerprinted assets returned
  `Cache-Control: public, max-age=31536000, immutable`; direct
  `/css/main.css` returned `Cache-Control: public, max-age=3600`.
- Packaged asset sizes included `main.css` at 123,780 bytes,
  `command-center.css` at 6,273 bytes, `shared-folder.css` at 11,341 bytes,
  `void-discovery.css` at 5,190 bytes, and `app.js` at 1,934 bytes.
- The preliminary 390 by 844 Void Explore check reported no horizontal overflow and visibly
  rendered the responsive discovery-card layout.
- Browser console warning/error inventory was empty.
- After merge, production root, liveness, readiness, `robots.txt`, and
  `sitemap.xml` returned status code 200 OK locally; public root, `robots.txt`,
  and `sitemap.xml` also returned status code 200 OK.
- Production rendered asset namespace `/36f78760656a39a24c90/`, exactly matching
  the canonical merged Git-tree static fingerprint. The isolated worktree value
  `/520a074536dc2f6e737d/` correctly differed because that checkout contained
  different line-ending bytes; each namespace fingerprints the bytes its build
  actually serves.
- The public Login page contained only `app.js`, `auth/login.js`, and Bootstrap
  script tags; it contained no Blog, Gallery, or player implementation script.
- The public Command Center shell preserved feature-before-shared stylesheet
  order. Its fingerprinted CSS and JavaScript returned one-year immutable cache
  headers, while direct `/css/main.css` retained the one-hour cache.

## Pass / Fail

Pass. The browser phase meets its delivery budget, demand-loading, stylesheet
ownership, deterministic fingerprint, cache, responsive-layout, automated-test,
runtime-isolation, and cleanup requirements.

The complete fresh gate ran `:website:check --rerun-tasks` successfully in
3 minutes 12 seconds across 19 Gradle tasks. It included 311 JavaScript tests
with 0 failures and 1,617 Java tests with 0 failures, 0 errors, and 3 intentional
skips.

## Evidence

- Deterministic build fingerprint:
  `520a074536dc2f6e737d` on two repeated builds.
- Backend-only probe fingerprint:
  `520a074536dc2f6e737d`.
- Static-file probe fingerprint:
  `44aff8b36767e52d0676`.
- Fingerprint after removing the static probe:
  `520a074536dc2f6e737d`.
- The boundary-shift verification failed under the old unframed serialization
  and passes with fixed-size per-file content digests.
- Initial graph calculation: 64,123 raw bytes across 10 modules.
- Focused delivery configuration suite: 8 tests passed.
- JavaScript suite: 311 tests passed.
- Full website gate: `BUILD SUCCESSFUL in 3m 12s`, 19 tasks executed.
- Candidate startup log: Tomcat started on port 8093 and application startup
  completed in 5.768 seconds.
- Cleanup proof: `DATABASE_BEFORE=true`, `DROP_OK=1`,
  `DATABASE_AFTER=false`; port 8093 free; production PID 39812 unchanged.
- PR #1337 required checks: Linux, macOS, Windows, dependency review, and all
  CodeQL analyses passed at head `c530fe3279eaab6ee047a424b7dda6190fd14832`.
- Post-merge runs: CI Build `30729190961` and CodeQL `30729190934` passed at
  merged SHA `95d805658beaa4c62a8b5e56af9bbf1c0aca66a6`.
- Production listener rotated from PID 39812 to PID 33336. MongoDB,
  ChristopherBellDev, and cloudflared remained Running/Automatic.
- Canonical merged Git-tree fingerprint and production namespace:
  `36f78760656a39a24c90`.

## Bugs / Follow-ups

No browser-delivery defect was found. Authenticated visual exercise of Command
Center, Shared Folder, Music, and player interactions was intentionally not
performed because the disposable runtime contained no privileged fixture account.
Their stylesheet ownership, access-loss behavior, media loader behavior, and
responsive contracts are covered by the focused automated suites; the public
runtime checks covered the delivery boundary affected by this phase.
