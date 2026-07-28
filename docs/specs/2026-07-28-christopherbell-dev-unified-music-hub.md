# christopherbell.dev Unified Music Hub

## Document Status

Ready for review.

## Purpose

Build one native Music Hub for `christopherbell.dev` that turns audio below
`A:\Shared\Music` into a fast, indexed library with a shared smart radio,
playlists, a shared queue, listening history, direct metadata editing, and
seamless site-wide playback.

The Music Hub must remain part of the existing website rather than introducing
Jellyfin, Navidrome, or another service and authentication silo.

## Background

The website already exposes an authenticated Shared Folder, a persistent
site-wide media player, and one shared radio timeline. The current file browser
is useful for raw storage operations but is not an album-, artist-, or
playlist-oriented music library. Radio selection has limited preference data,
and audio metadata is primarily discovered by the browser while playing a file.

The desired result is a dedicated `/music` experience with its own permissions.
It should be the main place for listening and managing music while Shared Folder
remains the raw file-management and download surface.

## Goals

- Provide one cohesive Music Hub for library browsing, playlists, queue, radio,
  history, favorites, exclusions, metadata repair, and playback.
- Keep one server-owned shared radio timeline and one shared administrative
  queue rather than per-user stations.
- Keep active music playing without interruption while navigating anywhere in
  the same browser tab.
- Introduce independent `MUSIC_READ` and `MUSIC_WRITE` capabilities.
- Allow safe, reversible edits to metadata embedded in the original media file.
- Make Music visible in the main navigation while auditing denied access.
- Extend normal browser login usability with a bounded, revocable session model.
- Preserve production safety on the Windows host and avoid background work with
  unbounded CPU, memory, process, storage, or database cost.

## Non-Goals

- Do not introduce a separate media server, user database, or embedded third-
  party media interface.
- Do not create personal radio stations, personal playlist silos, or per-user
  copies of the music catalog.
- Do not grant music downloads through `MUSIC_READ` or `MUSIC_WRITE`.
- Do not make metadata editing available for formats that cannot be rewritten
  and validated without re-encoding audio.
- Do not promise uninterrupted audible playback across a browser refresh when
  browser autoplay policy requires a new user gesture.
- Do not change explicit API bearer-token lifetime or renewal behavior as part
  of the browser-session work.

## Authorization Model

### Capabilities

- `MUSIC_READ` grants access to `/music`, catalog browsing and search, audio
  streaming, manual playback, shared-radio listening, playlists, queue state,
  and history. It never grants a download.
- `MUSIC_WRITE` implies `MUSIC_READ`. It permits shared playlist changes, queue
  changes, favorites, radio exclusions, metadata and artwork edits, backup
  restoration, and other Music Hub management actions. It does not grant a
  download.
- `SHARED_FOLDER_READ` and `SHARED_FOLDER_WRITE` remain independent. They do not
  imply either Music capability, and Music capabilities do not imply either
  Shared Folder capability.
- A user who has both `MUSIC_READ` and `SHARED_FOLDER_READ` may still download a
  music file through the Shared Folder because the broader Shared Folder grant
  is additive.
- `ADMIN` has effective `MUSIC_READ` and `MUSIC_WRITE` by default.
- Removing `MUSIC_READ` from an account also removes `MUSIC_WRITE`.
- Back Office allows admins to grant and revoke both Music capabilities.

### Enforcement

Every catalog, stream, radio, playlist, queue, preference, history, artwork,
metadata-edit, and restore boundary must enforce the relevant Music capability
server-side. Hiding a button is not authorization. Music streaming must use a
Music-specific content boundary that cannot be converted into an attachment or
download by changing query parameters or headers.

Existing Shared Folder endpoints keep their current authorization contract.
Possessing a Shared Folder capability does not unlock `/music` without an
explicit `MUSIC_READ` grant.

## Navigation and Denied Access

- Music is a top-level navigation item visible to anonymous and authenticated
  visitors.
- An anonymous `/music` request renders a polished sign-in/access-required page.
- An authenticated account without `MUSIC_READ` renders a polished access-
  denied page.
- Access-denied rendering must not disclose catalog contents, paths, artwork,
  station state, account data, or authorization internals.

### Music Access Audit

Back Office includes an admin-only Music Access page with bounded filtering by:

- date range;
- allowed or denied outcome;
- authenticated or anonymous actor;
- account identity; and
- trusted client IP.

Logged-in denied attempts record the durable account identity. Anonymous denied
attempts record the client IP only after the existing trusted-proxy resolution
boundary has rejected spoofable forwarding data. Repeated attempts by the same
actor and route are aggregated within a short bounded window and retain an
attempt count, first occurrence, and latest occurrence rather than creating an
unbounded row per request.

Anonymous IP records expire after 30 days. Expiration is server-enforced and
covered by cleanup tests. Audit persistence failure never grants access; a
request that should be denied remains denied.

## Music Catalog

### Source and identity

The catalog indexes ordinary audio files recursively below `A:\Shared\Music`.
Each catalog record is bound to the existing stable file observation identity
and source revision so stale metadata cannot authorize or overwrite a changed
file.

Expected catalog fields include:

- safe relative path and display filename;
- title, artist, album artist, album, track number, disc number, genre, and year;
- duration and audio/container format information;
- embedded artwork identity and bounded thumbnail references;
- file size, modified time, stable observation token, and catalog status; and
- extraction or edit problems safe for administrative display.

Missing metadata uses deterministic filename and folder fallbacks without
inventing embedded values.

### Index maintenance

- Shared Folder uploads, moves, restores, metadata edits, and deletes notify the
  catalog immediately after the visible filesystem transition succeeds.
- A bounded incremental reconciliation detects files changed directly on the
  drive.
- Unchanged revisions are not reopened, rehashed, or re-probed.
- Probe concurrency, process lifetime, output size, artwork size, and scan work
  per cycle are bounded.
- Removed files leave active results promptly. Missing or failed files do not
  stop the rest of the catalog from serving.

MongoDB is the browsing index, not the media source. Streaming always rechecks
the current filesystem revision at the existing safe read boundary.

## Music Hub Interface

`/music` opens on a unified Listen dashboard containing:

- current shared-radio artwork, title, artist, album, elapsed time, and status;
- a prominent Listen Live action;
- the upcoming shared queue;
- recently played tracks; and
- quick access to favorites and playlists.

Primary sections are:

- **Library**: Albums by default, with Artists, Songs, Genres, and Folders as
  alternate views.
- **Playlists**: shared playlists managed by `MUSIC_WRITE` holders.
- **Queue**: the current shared queue with reorder and removal controls for
  `MUSIC_WRITE` holders.
- **History**: searchable shared-radio and playback history.
- **Manage**: writer-only metadata problems, excluded tracks, failed files,
  edits, private backups, and undo controls.

Search remains visible while navigating and matches title, artist, album,
genre, playlist, and safe relative path. Mobile selection opens the chosen
album or track near the top of the viewport rather than below a long result
list. Writer actions use compact overflow menus on small screens.

## Persistent Playback Handoff

The existing persistent media session remains the single owner of the live
audio element in a browser tab.

- While the user is on `/music`, the compact bottom player is hidden and the
  Music Hub renders a full player bound to the same live session.
- Navigating away collapses the full player into the bottom bar without
  replacing the audio element, changing the source, seeking, pausing, or losing
  buffer, volume, queue, or radio state.
- Returning to `/music` hides the bottom bar and reconnects the expanded player
  to the same live session without restarting playback.
- The handoff remains same-tab only.
- A true page refresh restores the saved source and position. Audible autoplay
  after refresh remains subject to browser policy and may require a tap.

The expanded and compact players are control surfaces over one playback owner,
not two synchronized media elements.

## Shared Smart Radio and Queue

### Station ownership

The server owns one durable station timeline. It advances independently of
listeners by using durations extracted into the server-side catalog. Joining a
station must return the elapsed point on the current timeline rather than start
a new track for that listener.

### Smart rotation

Eligible selection must:

- reject every radio-excluded track;
- give favorites a strong but bounded weight;
- enforce separate recent-track and recent-artist cooldowns;
- prefer eligible music that has gone longest without being heard;
- retain an occasional random eligible selection; and
- avoid immediate repetition whenever another eligible track exists.

Selection remains deterministic under injected time and randomness for tests.
One shared history event is recorded per station transition, not per connected
listener.

### Queue behavior

`MUSIC_WRITE` holders may Play Next, Add to Queue, reorder, or remove queued
tracks. Queue items override smart selection for everyone in their explicit
order. Smart rotation resumes automatically when the queue becomes empty.

Missing, changed, unreadable, unsupported, or failed queued tracks are skipped
and recorded for writer review instead of stopping the station. Queue mutation
uses revision and concurrency controls so simultaneous writers cannot silently
overwrite each other.

## Preferences, Playlists, and History

- A favorite is a shared positive signal that increases but does not guarantee
  radio selection.
- Exclude from Radio is a shared hard rule.
- There are no star ratings or per-user recommendation profiles.
- Playlists are shared and writable only with `MUSIC_WRITE`.
- Readers may browse and play playlists but cannot mutate them.
- Radio history is shared and ordered by server transition time.
- Manual playback history must be bounded and must not create one database write
  per media progress event.

## Direct Metadata Editing

`MUSIC_WRITE` holders may edit title, artist, album artist, album, track and disc
numbers, genre, year, and embedded artwork for supported formats.

Each edit must:

1. authorize `MUSIC_WRITE` and validate CSRF for browser mutations;
2. recheck the exact visible file revision;
3. create a checksum-bound original backup in private storage with a 30-day
   expiration;
4. copy or open the source through the existing safe filesystem boundary;
5. rewrite metadata in private staging without re-encoding the audio stream;
6. probe the staged output and verify container, audio codec/stream, duration,
   requested tags, and bounded artwork;
7. atomically replace the visible source only if its revision is still current;
8. refresh the catalog after the replacement succeeds; and
9. record success or a safe failure in the administrative audit history.

Unsupported formats remain playable and show read-only metadata controls. A
failed edit leaves the visible source untouched.

Undo is `MUSIC_WRITE`-only and revision-checked. It restores a selected private
backup without overwriting a file changed after the corresponding edit. Restore
must itself use staged validation and an atomic visible transition. Backup
expiration is audited and must not follow links or cross the private backup
root.

## Browser Session Lifetime

The longer lifetime applies only to normal browser-cookie sessions. Explicit
API bearer-token behavior remains unchanged.

- Browser sessions have a seven-day inactivity timeout.
- Browser sessions have a 30-day absolute lifetime from authentication.
- Eligible interactive browser activity may rotate and extend the cookie up to
  the absolute deadline.
- Media streaming, range requests, radio polling, background API refreshes,
  health checks, static assets, and other non-interactive traffic never refresh
  session activity or lifetime.
- Tokens rotate through a server-tracked session identity. Old rotated tokens
  receive a bounded overlap only when necessary for concurrent browser requests
  and then become invalid.
- Logout, password reset, account suspension, account deletion, and sensitive
  role or permission changes revoke affected browser sessions immediately.
- Cookies remain Secure, HttpOnly, SameSite-protected, path-scoped as
  appropriate, and absent from URLs, response bodies, and logs.
- Session creation, renewal, revocation, expiration, and invalid reuse produce
  safe security audit events without logging token material.

## Resource and Failure Boundaries

- Catalog scans, media probes, tag-edit jobs, thumbnails, queue/history growth,
  access audit growth, backups, and cleanup work all have explicit bounds.
- Full embedded artwork is not repeatedly loaded for list views; bounded cached
  thumbnails serve browsing surfaces.
- Media tools run only through the existing pinned, restricted worker boundary,
  never from request parameters or an unrestricted website process.
- Failure to read one file cannot make the whole library unavailable.
- Failure to persist a preference or queue mutation is visible to the writer and
  never reported as success.
- Session-store failure requires reauthentication rather than silently
  extending access.
- Access-audit failure never changes an authorization decision.
- Absolute filesystem paths, private backup paths, worker commands, token
  material, and internal exception details never reach browser responses.

## Expected Ownership Areas

Implementation is expected to touch focused ownership areas in the
`christopherbell.dev` spoke repository, including:

- account permissions, effective authority evaluation, browser session
  issuance/revocation, and Back Office account management;
- shared-folder safe read/mutation boundaries and media worker contracts;
- a new focused music catalog/radio/playlist/queue/history/editing package;
- Music and Back Office controllers, request/response models, and repositories;
- main navigation and `/music` templates;
- the persistent site media player, expanded Music player, Music page modules,
  and responsive CSS;
- production worker/install configuration and bounded cleanup tasks; and
- package documentation, operations guidance, migrations, and tests.

Exact files and literal line ranges belong in the reviewed implementation plan
after inspecting refreshed `origin/main`.

## Validation Plan

### Automated behavior

- Test every effective combination of `MUSIC_READ`, `MUSIC_WRITE`,
  `SHARED_FOLDER_READ`, `SHARED_FOLDER_WRITE`, role hierarchy, revocation, and
  admin defaults.
- Prove music-only readers can stream but cannot download through alternate
  routes, headers, ranges, content dispositions, or endpoint substitution.
- Test anonymous and authenticated denied-access rendering and auditing,
  trusted-proxy IP resolution, spoofed forwarding rejection, aggregation,
  Back Office filtering, bounds, authorization, and 30-day expiration.
- Test incremental indexing, file revision changes, stale rows, missing tags,
  unsupported files, probe failures, artwork bounds, and direct-drive changes.
- Test smart selection weights and hard rules with injected clock and randomness,
  listener-free advancement, queue precedence, history uniqueness, concurrent
  mutation, restart recovery, and missing queued tracks.
- Test tag edits against representative supported media fixtures, including
  concurrent source changes, worker failure, invalid output, atomic replacement,
  undo, expiration, and unchanged audio-stream evidence.
- Test seven-day idle expiration, 30-day absolute expiration, rotation overlap,
  non-interactive request exclusion, logout, password reset, suspension,
  deletion, permission-change revocation, and unchanged API bearer behavior.
- Test the compact/expanded player as two views over one media element and prove
  no pause, source reload, seek, or state loss across repeated navigation.

### Runtime acceptance

- Run the complete Java and browser test suites and all shared-folder worker and
  operations checks required by the repository.
- Validate on a non-8080 port on the Windows production host with isolated test
  data and the production-compatible profile.
- Exercise desktop and mobile Music Hub navigation, library search, radio,
  queue, playlists, tag editing, undo, Back Office access logs, permission
  changes, and session renewal/expiration flows.
- Verify continuous playback while navigating between `/music`, public pages,
  Shared Folder, Messages, Tools, and Back Office.
- Publish through a reviewed PR, require platform CI, Dependency Review, and
  CodeQL success, merge to `main`, and let the automatic production pipeline
  deploy.
- Confirm the deployed release SHA, root route, liveness, readiness, authorized
  Music behavior, denied Music behavior, and unchanged Shared Folder behavior.

## Acceptance Criteria

- Music is visible in the main navigation to everyone.
- Only accounts with effective `MUSIC_READ` can enter or stream from the Music
  Hub.
- `MUSIC_READ` and `MUSIC_WRITE` alone cannot download music.
- `MUSIC_WRITE` implies `MUSIC_READ`; admins receive both; Shared Folder
  capabilities remain independent.
- Back Office can grant/revoke Music capabilities and inspect bounded Music
  access logs.
- Anonymous denied attempts retain trusted IP evidence for no more than 30 days.
- `/music` provides the approved Library, Playlists, Queue, History, Manage, and
  expanded-player experience on desktop and mobile.
- Audio continues through same-tab site navigation with an uninterrupted
  compact-to-expanded player handoff.
- One shared smart station advances without listeners, respects favorites,
  exclusions, cooldowns, randomness, queue overrides, and missing-file handling.
- Direct metadata edits preserve audio, create 30-day undo backups, validate
  staged output, replace atomically, and never overwrite a changed source.
- Browser sessions honor seven-day inactivity and 30-day absolute limits without
  renewal from media/background traffic; revocation events take effect
  immediately.
- All required automated, alternate-port, CI, deployment, and production smoke
  evidence passes before completion is claimed.

## Open Questions

None. Product and security decisions required for implementation planning are
resolved in this specification.
