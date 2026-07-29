# 2026-07-29 - Social Relationship Feed Scalability Issues 1273-1279 Completed

## Request

Complete every open `azurras/christopherbell.dev` issue #1258-#1307 without routine approval pauses, preserving the dirty authoritative checkout and carrying each batch through tests, PR/CI, merge, production verification, issue closure, and Builder evidence.

## Work Completed

Batch 3 moved likes and follows into deterministic unique edge collections, added retry-safe desired-state mutations, assembled feed engagement in constant query counts, filtered visibility before cursor limits, bounded legacy account histories, removed repair writes from GET paths, and made root reply metrics/expiration propagation atomic or bulk. PR #1323 passed every platform/security check and squash-merged as `e3afbf3c9eeb65525f573f299f82287ef8665554`; issues #1273-#1279 closed automatically.

## Decisions

Relationship edges are presentation truth. Embedded post counters remain bounded expiration bookkeeping. Compatibility toggle/history APIs remain available but edge-backed, capped, and explicitly deprecated. Migrations copy before removing arrays and stop startup on incomplete state.

## Validation

Final `:website:check` passed 1,431 Java tests with zero failures/errors and 3 skipped plus frontend/package/sensor/policy gates. Port-8093 runtime acceptance covered V009/V010, retries, concurrency, capacity, pure reads, pagination, and cleanup. Production rotated to PID 60136, served exact merge-SHA assets locally/publicly, reported liveness/readiness `UP`, applied migrations 009/010, created required indexes, removed legacy arrays, and backfilled all root metrics.

## Current State

The authoritative checkout remains untouched. The campaign ledger is active with 28 issues remaining: #1280-#1307.

## Follow-ups

Inspect refreshed merged main for WFL issues #1280-#1289, save/review the Batch 4 plan, and repeat the full delivery loop.
