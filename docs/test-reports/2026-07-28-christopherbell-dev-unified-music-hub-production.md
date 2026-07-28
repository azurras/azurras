# ChristopherBell.dev Unified Music Hub Production Test Report

## Document Status

complete

## Story/Issue

Deliver the approved unified Music hub for `christopherbell.dev`: independent Music permissions, protected catalog and playback, global radio/library state, metadata editing, durable browser sessions, and a responsive persistent player.

## Branch

- Spoke branch: `codex/unified-music-hub`
- Pull request: [azurras/christopherbell.dev#1312](https://github.com/azurras/christopherbell.dev/pull/1312)
- Production commit: `baf8910dd6707260ec02af94c13d36c1eb2d6979`

## App / Environment

- App: `christopherbell.dev` Spring Boot website
- Environment: native Windows production host, public Cloudflare tunnel
- Public base URL: `https://www.christopherbell.dev`
- Local listener: port `8080`
- Data/services: production MongoDB, protected `A:\Shared\Music`, pinned FFmpeg/FFprobe tool manifest

## Local Run Details

The existing push-to-main deployment workflow built and started the merged release non-interactively through the `ChristopherBellDev` Windows service. The application was left running. At verification time, `ChristopherBellDev`, `MongoDB`, and `cloudflared` were all `Running` with `Automatic` startup. Port `8080` had one listener owned by PID `38612`, created at `2026-07-28 13:20:05` America/Chicago. No manual service rotation or ACL change was performed.

## Test Cases

1. Load the public home page and confirm the deployed versioned asset path identifies the merged commit.
2. Load the public `/music` shell and its versioned stylesheet.
3. Query the public Music access endpoint anonymously and confirm a data-free denial response.
4. Attempt anonymous catalog, radio, and track-stream requests and confirm protected data remains inaccessible.
5. Inspect Content Security Policy, HSTS, and no-store response headers.
6. Verify the production services and single live listener.

## Data Sent

- `GET https://www.christopherbell.dev/`
- `GET https://www.christopherbell.dev/music`
- `GET https://www.christopherbell.dev/api/music/2026-07-28/access`
- `GET https://www.christopherbell.dev/api/music/2026-07-28/catalog`
- `GET https://www.christopherbell.dev/api/music/2026-07-28/radio`
- `GET https://www.christopherbell.dev/api/music/2026-07-28/tracks/00000000-0000-0000-0000-000000000000/stream`
- `GET https://www.christopherbell.dev/baf8910dd6707260ec02af94c13d36c1eb2d6979/css/music.css`
- No cookies, credentials, request bodies, or mutation requests were sent.

## Response Received

The running app returned these HTTP responses; public Music shell status code: 200.

- Home: HTTP `200`; versioned main stylesheet referenced commit `baf8910dd6707260ec02af94c13d36c1eb2d6979`.
- Music shell: HTTP `200`.
- Music stylesheet: HTTP `200`, 7,979 bytes.
- Access probe: HTTP `200`, body `{"authenticated":false,"allowed":false,"canManage":false,"reason":"SIGN_IN_REQUIRED"}`.
- Catalog, radio, and stream: HTTP `403` for the anonymous caller.
- Access probe: `Cache-Control: no-store`.
- Music shell: CSP present with `default-src 'self'`, `object-src 'none'`, `frame-ancestors 'self'`, and `media-src 'self' blob:`; HSTS present with one-year max age and subdomains.
- Services: `ChristopherBellDev`, `MongoDB`, and `cloudflared` all `Running` / `Automatic`.
- Listener: one `::`:8080 socket owned by PID `38612`.

## Pass / Fail

All six production smoke cases passed. The exact merged release is public, anonymous users receive only the access shell/status response, protected Music data and media are denied, security headers are active, and the service stack is healthy.

## Evidence

- Local full repository `:website:check`: passed before push.
- JavaScript suite: 246 tests passed.
- Windows production Pester suite: 243 passed, 0 failed, 4 machine/elevation-dependent skips.
- Pull request CI: Ubuntu, macOS, Windows, dependency review, and both CodeQL language analyses passed.
- CodeQL found one case-insensitive test-safety gap during review; commit `ce19fb33` corrected it before merge and the final scan passed.
- Cross-platform process tests were corrected in commits `18b47f64` and `b6b0310b`; the final three-platform matrix passed.
- Production smoke was executed after the public version path changed to the exact merge commit.

## Bugs / Follow-ups

No release-blocking defect remains. Authenticated reader/writer/admin production interactions and desktop/mobile visual playback were not automated in this final smoke because no authenticated browser-control session was available. Those authorization, mutation, player handoff, and responsive-layout paths are covered by focused Java and JavaScript tests; a normal signed-in usage pass remains useful observational confirmation.
