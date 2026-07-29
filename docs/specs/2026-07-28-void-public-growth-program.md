# Void Public Growth Program

## Document Status

ready-for-execution

## Purpose

Turn Void into the public reason to visit and join `christopherbell.dev`: a chronological, low-pressure social network where every post begins with a limited lifespan and the community decides what survives.

The program will ship as three production releases:

1. Relaunch Void around its existing survival mechanic.
2. Add public discovery without popularity ranking.
3. Connect opted-in accounts to the wider social web through ActivityPub.

Each release must be complete, secure, deployed, and production-verified before implementation begins on the next release.

## Background

Void already provides the essential product foundation:

- Anonymous users can read the public feed, profiles, threads, and individual post pages.
- Anyone can create an account and authenticated users can post, reply, follow, and like.
- A root thread begins with a 24-hour lifespan.
- Each keep-alive interaction and each reply adds 24 hours to the entire thread.
- Removing a keep-alive removes its 24-hour extension.
- Replies inherit the root thread expiration.
- The browser already renders a live lifespan countdown and removes expired posts.
- Public post routes are shareable, but their social metadata is generic.
- The feed includes a popularity-derived `Active` sort, which conflicts with the approved no-popularity-ranking identity.

The growth problem is not a lack of unrelated features. Void needs a memorable public proposition, a way for strangers to discover active conversations, and distribution beyond the existing site.

## Product Identity

Void's public proposition is:

> Nothing lasts unless people care. Every post begins with 24 hours. Keep it alive or let it disappear.

The experience has four permanent principles:

- The primary feed is chronological, not algorithmically ranked.
- Public popularity scores never determine ordering or recommendation.
- A keep-alive is functional: it adds 24 hours to the thread.
- A reply is participation and therefore also adds 24 hours to the thread.

The public feed may show a quiet keep-alive count because it explains the remaining lifespan. The count must not become a leaderboard or ranking input.

## Goals

- Give a first-time visitor a clear explanation of Void within one screen.
- Make the survival mechanic visible, understandable, and satisfying to use.
- Keep public reading open and account creation immediate.
- Give every active thread an attractive, meaningful share experience.
- Help strangers find new, fading, revived, and topic-relevant conversations.
- Recommend people through shared interests rather than follower or like totals.
- Allow opted-in Void accounts to participate in the ActivityPub network.
- Preserve temporary local content semantics and clearly disclose federation limits.
- Add proportionate abuse, moderation, privacy, and operational controls before increasing reach.

## Non-Goals

- No engagement-ranked or personalized infinite-scroll algorithm.
- No advertising, monetization, premium tiers, or creator payouts.
- No public leaderboards for likes, keep-alives, followers, or post survival.
- No guarantee that a federated remote server deletes its cached copy of expired content.
- No federation of private messages, Music, Shared Folder content, administrative data, or private account information.
- No simultaneous implementation of all three releases.

## Release Sequence

Release status as of July 29, 2026:

- Release 1 - Keep Alive Relaunch: complete and production-verified at merge `7e958e737b34563d6d49a078243437d5fa9e3377` through PR `azurras/christopherbell.dev#1314`.
- Release 2 - Public Discovery: complete and production-verified at merge `f77c5f5bb644cc75cf98b27e722efdc00cd036f1` through PR `azurras/christopherbell.dev#1315`.
- Release 3 - ActivityPub Federation: discovery foundation, controlled-peer outbound delivery, and production read-only discovery activation are complete and production-verified through PRs `azurras/christopherbell.dev#1316`, `#1317`, and `#1318`, latest merge `8405cd77d0f1743fe33d70cc80b47e37048090a0`. Discovery is live; inbound and outbound remain disabled until their separate controlled interoperability gates pass.

### Release 1: Keep Alive Relaunch

#### Public experience

The `/void` hero must state the survival proposition and explain three rules:

1. Every thread starts with 24 hours.
2. Each keep-alive adds 24 hours.
3. Each reply adds 24 hours to the whole thread.

Anonymous visitors continue to read without an account. The composer prompt must offer immediate signup or login and preserve the exact feed, thread, or post destination across authentication.

The existing Like action becomes **Keep alive · +24h**. After the current user acts, its selected state becomes **Kept alive**. The quiet numeric count remains visible. The control must retain an accessible name that explains the 24-hour effect.

Each active root post and reply context must show the server-derived live countdown. A confirmed keep-alive produces a restrained `+24h` visual update; the UI must not extend time optimistically before the server response. Undoing a keep-alive restores the unselected state and server-derived expiration.

Reply composers must explain that a successful reply adds 24 hours to the entire thread. After a confirmed reply, the root and every visible reply must display the updated shared expiration.

Every post retains Copy link and gains a native Share action when `navigator.share` is available. The fallback copies the canonical post URL.

The popularity-derived `Active` sort is removed. Release 1 retains newest-first and expiring-soon ordering. Recently revived ordering belongs to Release 2 and must use extension time rather than totals.

#### Public post previews and expiration

An active `/p/{postId}` response must provide server-rendered, escaped social metadata containing:

- The author's public username.
- A bounded plain-text excerpt.
- The canonical post URL.
- A recognizable 1200x630 Void preview image.
- A concise explanation that the thread is temporary.

The preview must never include raw HTML, secrets, private profile data, or content from an expired thread. An expired or missing public post renders a deliberate `This post vanished into the Void` state with a route back to `/void`; it must not expose deleted content through metadata, APIs, caches, or logs.

#### Server and compatibility

The current 24-hour base and extension calculations remain authoritative and unchanged. Existing Like API routes may remain as compatibility boundaries, while new UI and documentation use Keep alive terminology. A future versioned API may rename the action without breaking current clients.

Failure behavior:

- Anonymous interaction redirects to login and preserves the intended return URL.
- A rejected, rate-limited, or failed keep-alive leaves the prior rendered state intact and shows an actionable message.
- A thread that expires during interaction returns the existing not-found domain result and disappears cleanly.
- Malformed or absent expiration data fails closed to server repair behavior; the browser does not invent a lifespan.

#### Release 1 acceptance criteria

- A first-time anonymous visitor can explain the 24-hour survival mechanic from `/void` without interacting.
- Active threads show a ticking countdown and quiet keep-alive count.
- Keep alive adds exactly 24 hours after server confirmation; undo removes that extension.
- A reply adds exactly 24 hours to the root and all descendants.
- No feed option ranks by keep-alive or reply totals.
- Shared active posts have meaningful escaped metadata; expired posts disclose no original content.
- Public reading, signup, login return paths, reporting, deletion, following, and existing thread navigation continue working.

### Release 2: Public Discovery

#### Explore surface

Add a public `/void/explore` page composed of independently loaded, failure-isolated sections:

- **New arrivals:** active root posts ordered by creation time descending.
- **Fading soon:** active root posts ordered by expiration ascending.
- **Recently revived:** active root posts ordered by the most recent confirmed lifespan extension.
- **Topics:** active hashtags ordered by recent active-post or extension time, never total likes.

Each section uses bounded cursor pagination with stable unique tie ordering. A failed section must not prevent other sections from rendering. Empty sections explain their selection rule.

#### Revival state

Add a dedicated nullable `lastExtendedOn` timestamp to root threads. It changes only when a confirmed keep-alive or reply adds lifespan. Editing, viewing, sharing, following, and notification delivery must not update it.

Undoing a keep-alive recalculates the expiration but does not create a new revival event. The implementation must define and test how `lastExtendedOn` is repaired for historical threads; the approved default is to leave historical values null until the next genuine extension.

#### Topics

Topics are hashtags parsed from post text at the trusted post-write boundary.

- Normalize Unicode safely and store a canonical lowercase lookup form plus display text.
- Permit at most five unique topics per post.
- Bound each topic to 40 Unicode code points after normalization.
- Reject or ignore malformed topics without treating the rest of a valid post as HTML.
- Public topic pages use canonical encoded routes such as `/void/topic/music`.
- Topic feeds are chronological and include only active public threads.

Topic pages and API responses must escape all user-controlled display values. Topic lookup must not construct raw regular expressions or unbounded database queries from input.

#### Account discovery

Signed-in users receive **People you may want to follow** based on deterministic overlap among topics they have:

- Posted in.
- Replied to.
- Kept alive.

Follower count, keep-alive count, post lifespan, and global popularity are not inputs. Already-followed, blocked, hidden, deleted, and self accounts are excluded. Ties use recent public activity and then a stable unique account identifier.

Anonymous visitors see a daily deterministic rotation of recently active public accounts. This prevents a permanent popularity hierarchy while keeping the section stable within a visit.

#### Abuse and moderation

- New accounts receive stricter bounded posting, reply, follow, and keep-alive limits until the configured account-age threshold passes.
- Existing report, hide-thread, block, and administrative moderation boundaries remain authoritative.
- Explore endpoints return only the fields required for public rendering.
- Discovery caches exclude expired posts and have TTLs shorter than the expiration-cleanup interval.
- Topic and account queries require appropriate indexes and bounded results.

#### Release 2 acceptance criteria

- All four Explore sections implement their stated non-popularity ordering.
- Recently revived changes only after a confirmed lifespan extension.
- Topic extraction is normalized, bounded, deduplicated, safe, and covered across Unicode and malformed input.
- Suggested accounts never use likes or follower totals and exclude disallowed relationships.
- Anonymous Explore access works without exposing protected account data.
- Partial backend failure produces localized UI errors rather than a blank page.

### Release 3: ActivityPub Federation

#### Consent and scope

Existing accounts are not federated until their owners explicitly enable federation. New signup includes an explicit **Federate my public Void posts** choice that is enabled by default and accompanied by a clear disclosure:

- Federated posts and interactions are delivered to independent servers.
- Void sends Delete activities when content expires.
- Remote servers may retain cached or copied content despite deletion.

Federation applies only to public Void identities, posts, replies, follows, and keep-alive interactions. Disabling federation stops new outbound delivery and inbound interaction for the account; it does not claim to retract remote history automatically.

#### Protocol surface

Implement the bounded ActivityPub surface needed for interoperable public accounts:

- `/.well-known/webfinger` discovery for `@username@christopherbell.dev`.
- NodeInfo discovery and metadata.
- Public ActivityPub actors.
- Per-actor inbox, outbox, followers, and following collections.
- A shared inbox for outbound fan-out efficiency.
- Content negotiation that preserves existing HTML profile and post routes.

The first interoperability target is current Mastodon-compatible behavior. Unsupported activity types return a safe protocol response and do not mutate local state.

#### Outbound mapping

- A new local root post or reply becomes a public `Create` containing a `Note`.
- A local reply uses the federated root or parent URI through `inReplyTo`.
- A local keep-alive becomes a `Like`; undo becomes `Undo(Like)`.
- Following a remote actor becomes `Follow`; local unfollow becomes `Undo(Follow)`.
- Expiration or owner/moderator deletion emits `Delete` with a tombstone to known recipients.
- Local edits may emit `Update` only while the post remains active.

Outbound delivery uses a durable queue with unique activity identifiers, per-recipient delivery state, bounded exponential backoff, terminal failure classification, and idempotent retry. One remote failure must not block other recipients or local requests.

#### Inbound mapping

- A verified `Follow` is accepted for an enabled public account.
- A verified `Like` on an active local thread adds 24 hours exactly once per remote actor and target.
- `Undo(Like)` removes that actor's active extension exactly once.
- A verified public `Create(Note)` reply adds 24 hours to the root thread after content and relationship validation.
- `Delete` removes or tombstones locally retained remote content.
- Duplicate or replayed activities do not add time or create duplicate objects.

Remote extensions are abuse-bounded:

- One active keep-alive per remote actor and local target.
- Bounded accepted replies per actor, domain, and time window.
- Per-actor and per-domain request limits.
- A configurable rolling per-domain lifespan-extension ceiling per local root thread.
- Blocked actors and domains contribute no lifespan and receive no delivery.

Local engagement retains the approved existing lifespan rules.

#### Federation security

- Verify supported HTTP signatures, content digests, request dates, actor ownership, and key identity before mutation.
- Reject stale timestamps, invalid algorithms, mismatched hosts, oversized bodies, duplicate activity IDs, and signature replays.
- Fetch remote actors and keys only through an outbound policy that blocks loopback, private, link-local, multicast, and cloud-metadata destinations after every DNS resolution and redirect.
- Bound redirects, response bytes, connect time, total time, content types, and concurrent remote fetches.
- Store per-account private signing keys encrypted at rest with a protected application secret and support deliberate rotation.
- Never log signing material, authorization data, raw signatures, cookies, or full private payloads.
- Separate inbound and outbound feature flags and kill switches.

#### Operations and moderation

The Back Office gains a Federation panel containing:

- Inbound and outbound enabled state.
- Queue depth, retry age, and terminal failures.
- Known and recently active domains.
- Actor and domain block controls.
- Per-domain failure summaries without raw post bodies.
- Separate inbound and outbound emergency kill switches.

Federation rollout is gated inside Release 3:

1. WebFinger, NodeInfo, and local actor discovery.
2. Outbound follows and post delivery to controlled test peers.
3. Outbound production delivery for opted-in accounts.
4. Inbound follows.
5. Inbound keep-alives and replies after signature, replay, SSRF, moderation, and lifespan evidence passes.

#### Release 3 acceptance criteria

- Existing accounts remain unfederated until opt-in; new signup displays the approved explicit consent.
- A Mastodon-compatible test peer can discover, follow, receive, and reply to an opted-in Void actor.
- Valid remote Like/Undo and reply activities produce exactly the approved lifespan changes once.
- Expiration emits bounded Delete delivery and removes the local thread.
- Signature, replay, payload, redirect, DNS/IP, rate-limit, block, and queue tests pass.
- Inbound and outbound kill switches stop their respective effects without taking down local Void.
- Back Office exposes actionable health and moderation state without secrets or post-body logging.

## Shared Architecture and Boundaries

Keep the following responsibilities separate:

- **Post lifespan domain:** calculates and persists local thread expiration and extensions.
- **Feed and discovery queries:** return bounded active public projections with explicit ordering.
- **Topic extraction:** validates and normalizes topics at write time.
- **Recommendation service:** computes bounded interest overlap without popularity inputs.
- **Federation protocol boundary:** parses, validates, signs, and serializes ActivityPub messages.
- **Federation delivery queue:** owns outbound effects, retries, and recipient state.
- **Federation inbox service:** owns idempotency, verification, mapping, and mutation dispatch.
- **Remote fetch policy:** owns SSRF-safe actor/key retrieval.
- **Moderation and operations:** owns blocks, feature flags, queue visibility, and kill switches.

The browser must not calculate authoritative expiration, discovery eligibility, federation identity, or moderation decisions.

## Expected Areas of Change

Exact files belong in per-release implementation plans, but expected modules include:

- Existing post expiration, interaction, creation, feed, thread, and view packages.
- Void feed rendering, controller, templates, CSS, and JavaScript tests.
- Social-preview rendering and public post/profile metadata.
- New discovery/topic packages and indexed Mongo projections.
- Account-follow and public-profile projections.
- New federation protocol, inbox, delivery, remote-fetch, persistence, and operations packages.
- Security configuration, rate-limit configuration, Back Office surfaces, operational documentation, and production configuration examples.

## Validation Strategy

Every release requires:

- RED-to-GREEN behavioral tests at the narrowest public boundaries.
- Java and JavaScript unit tests for all new invariants and malformed input.
- Security tests for anonymous/public versus authenticated mutation boundaries.
- Full repository checks and CodeQL/dependency review.
- Candidate validation on a non-8080 port on the Windows production host.
- One focused pull request per coherent release or release sub-stage.
- Automatic deployment from `main` followed by exact-SHA public smoke tests.
- Production verification that existing Void, login/signup, Messages, Music persistence, and administrative access remain intact.

ActivityPub additionally requires controlled-peer interoperability tests and negative tests for signatures, replay, SSRF, redirects, payload bounds, blocking, kill switches, idempotency, retry, and deletion.

## Rollback and Recovery

- Release 1 contains no required lifespan data migration and can roll back through the normal application release mechanism.
- Release 2 schema additions are additive. Old code ignores new topic and extension metadata; rollback must retain documents for later retry.
- Release 3 remains guarded by separate inbound/outbound flags. Disabling both returns Void to local-only operation without deleting queue evidence or signing keys.
- Federation queue replay must be deliberate and idempotent. Operators may retry terminal failures only through bounded Back Office actions.
- Expired local content is never restored merely because remote delivery failed.

## Success Measures

The site should track privacy-preserving aggregate product health rather than public popularity:

- Anonymous Void visits that reach an active post or thread.
- Signup completions that originated from Void.
- New accounts that post, reply, follow, or keep something alive within seven days.
- Share actions and successful copied links.
- Active threads with at least two distinct participants.
- Explore-to-thread and topic-to-thread navigation.
- Opted-in federated accounts, successful deliveries, remote follows, and valid remote interactions.

Metrics must be retained only as long as operationally useful, avoid raw post bodies and secrets, and never affect feed ordering.

## Open Questions

None. Product direction, lifespan rules, public access, discovery rules, consent model, and federation rollout order are approved.
