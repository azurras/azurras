# VIN Scheduling and Link Preview Issues 1176-1181 Spoke Update

- Status: complete
- Source repository: `https://github.com/azurras/christopherbell.dev.git`
- Reporting agent: Codex primary agent
- Related work: [Complete All Open christopherbell.dev Issues](../work/2026-07-25-complete-all-open-christopherbell-dev-issues.md)
- Implementation plan: [VIN, Scheduling, and Link Previews Issues 1176-1181](../implementation-plans/2026-07-26-vin-scheduling-link-previews-issues-1176-1181.md)
- Test report: [VIN Scheduling and Link Preview Issues 1176-1181](../test-reports/2026-07-26-vin-schedulers-link-preview-issues-1176-1181.md)
- Review: [VIN Scheduling and Link Preview Issues 1176-1181](../spoke-reviews/2026-07-26-vin-schedulers-link-preview-issues-1176-1181.md)
- Session memory: [VIN Scheduling and Link Preview Issues 1176-1181](../session-memory/2026-07-26-vin-schedulers-link-preview-issues-1176-1181.md)

## Result

Issues [#1176-#1181](https://github.com/azurras/christopherbell.dev/issues/1176) were
completed and closed through [PR #1257](https://github.com/azurras/christopherbell.dev/pull/1257).
The PR squash-merged to `main` as `9963ed0cc83f8b43f54612c1b8c6ed2966f22607`.

## Delivery

- Worktree: `A:\Projects\christopherbell.dev-worktrees\vin-schedulers-link-preview-1176-1181`
- Branch: `codex/vin-schedulers-link-preview-1176-1181`
- Final branch head: `c1e9fc4f660f2ed04dffe33d0d51a6b0e60d7958`
- Added versioned/expiring VIN cache records, ordered partial-success VIN batch decoding, validated
  scheduler configuration, renewable shared collector leases and durable run state, SSRF-safe
  bounded preview transport, success/failure preview caching, V003 indexes, and documentation.
- Hardening after focused tests covered null batch entries, per-input cache outages, lease release
  when status writes fail, overall-timeout cancellation of slow bodies, and bounded DNS resolution.

## Validation

- Final `:website:check` passed with 1,200 Java tests, zero failures or errors, three expected
  skips, JavaScript tests, packaged JAR, and sensor-runtime verification.
- Packaged acceptance on port `8092` used exact disposable database
  `christopherbell_batch7_20260726`. Public pages and batch behavior passed; protected state
  returned 403; V003 and its three indexes were inspected; the PID/database were removed.
- Ubuntu, macOS, Windows, Dependency Review, Actions, Java/Kotlin, JavaScript/TypeScript, and
  aggregate CodeQL all passed. PR #1257 had no comments, reviews, or unresolved threads.
- Native SYSTEM auto-deployment changed port 8080 from PID `41176` to `29164`. Local/external
  roots and `/vin-decoder` returned 200, ordered invalid/null batch results returned 200, 21 VINs
  returned 400, and protected state returned 403.
- Production V003 was `APPLIED` at `2026-07-26T11:27:46.286Z` with checksum
  `799e5a12c1bfc022217a2c9f1e29f50ed4eef9b7f03daba01121a90c696dbd32` and all expected indexes.
- `MongoDB`, `ChristopherBellDev`, and `cloudflared` are Running with Automatic startup.
- Live GitHub inventory returned no open issues.

## Blockers and Risks

No blocker remains. Authenticated collector contention and outbound preview requests were not
forced against live production; focused tests cover those concurrency and network-policy paths.
The dirty authoritative checkout was not modified.

## Next Action

None for this batch. Close the parent 58-issue Builder work record.
