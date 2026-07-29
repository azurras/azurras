# 2026-07-29 - Public SEO Accessibility Issues 1265-1272 Completed

## 10:27 - Public SEO Accessibility Issues 1265-1272 Completed

### Request

Complete every open `azurras/christopherbell.dev` issue #1258-#1307 without routine approval pauses, preserving the dirty authoritative checkout and carrying each batch through tests, PR/CI, merge, production verification, issue closure, and Builder evidence.

### Project Context

Builder is the workflow hub. Website changes use isolated worktrees from refreshed `origin/main`; production is the same native Windows host, and alternate validation uses a non-8080 port with both Mongo URI and database explicitly set. Batch 2 covers public SEO and accessibility issues #1265-#1272.

### Work Completed

Batch 2 centralized crawler policy, added true non-indexable unknown and dynamic 404s, rendered resource-specific metadata, generated a bounded data-backed sitemap, corrected WFL canonicals, repaired The Bell semantics and external-link safety, made button behavior explicit, and added standard auth form names/autocomplete/POST fallback. Independent review drove fixes for sitemap expiration/bounds, alias canonicals, credential fallback, and encoded protected-namespace handling.

PR #1321 passed all platform/security checks and squash-merged as `f31535f29312d24573a6031b0162aa8ebc4b5318`. Issues #1265-#1272 were closed after production verification.

### Decisions

Unknown browser-page GETs may reach MVC's true 404 renderer only when both raw and decoded request paths are outside protected namespaces; percent-encoded fallback paths are not public. Sitemap eligibility uses the same active/public and post-expiration policy as public rendering and caps pages at 50,000 URLs without scanning the corpus for impossible shards.

### Validation

Final `:website:check` passed 1,418 Java tests with zero failures/errors and 3 skipped plus all frontend/package/sensor/policy checks. The final JAR passed isolated port-8094 runtime acceptance and exact cleanup. Production rotated from PID 51060 to 46940, served merge-SHA assets, reached liveness/readiness UP, and passed external sitemap, 404/noindex, protected namespace, canonical, semantic markup, and auth form checks.

### Current State

The authoritative dirty checkout remains untouched. The campaign ledger is active with 35 issues remaining: #1273-#1307.

### Follow-ups

Inspect refreshed merged main for social-feed issues #1273-#1279, save and review the Batch 3 plan, and repeat the full delivery loop.
