# VIN Scheduling and Link Preview Issues 1176-1181 Spoke Review

## Document Status

complete

## Reviewed Spoke

- Repository: `https://github.com/azurras/christopherbell.dev.git`
- Worktree: `A:\Projects\christopherbell.dev-worktrees\vin-schedulers-link-preview-1176-1181`
- Branch: `codex/vin-schedulers-link-preview-1176-1181`
- Base: `abd2051e76155e5c01137ebec10c2d7550ec3556`
- Reviewed head: `c1e9fc4f660f2ed04dffe33d0d51a6b0e60d7958`
- Pull request: [#1257](https://github.com/azurras/christopherbell.dev/pull/1257),
  squash-merged as `9963ed0cc83f8b43f54612c1b8c6ed2966f22607`
- Issues: `#1176-#1181`

## Scope Reviewed

VIN cache lifecycle/versioning, ordered batch decoding, rate cost, typed scheduler configuration,
renewable distributed leases, durable collector outcomes, Canes manual contention, preview SSRF
policy, redirects/DNS/time/byte/content bounds, preview success/failure caching, migrations,
documentation, focused/full/runtime evidence, PR gates, production deployment, and issue closure.

## Findings

No remaining Blocker or Warning findings.

Review-driven testing found and corrected five boundary defects before publication: null batch
entries were copied through a null-rejecting collection helper; cache read failure could escape the
per-input result contract; initial/terminal collector status persistence failure could bypass exact
lease release; overall preview timeout could wait on slow body cleanup; and DNS resolution was not
initially charged to the same overall deadline. Focused regressions cover each corrected boundary.

## Validation Checked

- Final full repository gate: 147 Java suites, 1,200 tests, zero failures or errors, three expected
  skips; JavaScript, packaged JAR, sensor-runtime, and whitespace checks passed.
- Candidate JAR on port 8092 returned the specified ordered partial results and envelope failure,
  applied the exact V003 migration/indexes, then stopped cleanly with its database dropped.
- PR gates passed Ubuntu, macOS, Windows, Dependency Review, three CodeQL analyses, and aggregate
  CodeQL. GitHub exposed no comments, reviews, or unresolved review threads.
- Production rotated to PID `29164`, local/external public probes returned 200, new batch behavior
  passed, protected state returned 403, V003/indexes matched, and all services were healthy.
- All six issues closed and the live open-issue inventory is empty.
- `git diff --check` passed; the unrelated dirty authoritative checkout was not modified.

## House-Style Review

The implementation follows repository-native Java/Spring/Mongo conventions. Cache and lease state
make version, expiry, ownership, renewal, and terminal outcomes explicit. Batch effects preserve
input identity/order and keep item failures local. The preview transport validates every authority
before effects and bounds DNS, redirects, time, type, and bytes. Tests emphasize invariants and
failure boundaries rather than implementation-only call sequences.

## Risks

- Authenticated destructive collector execution was intentionally not forced against production;
  focused tests cover ownership loss, contention, persistence failures, and HTTP 409 behavior.
- Production link-preview checks used safe invalid inputs rather than contacting arbitrary public
  targets; the full destination matrix and bounded transport behavior are covered by tests.
- Protected SYSTEM release metadata remains inaccessible to the non-elevated shell by design;
  listener rotation, exact newly merged endpoint behavior, exact migration completion, external
  reachability, and service state prove the deployment without weakening ACLs.

## Requested Changes

None remaining.

## Merge Readiness

Complete. The final head passed review, full/runtime validation, all GitHub gates, squash merge,
native production acceptance, and issue closure.
