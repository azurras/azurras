# 2026-08-02 - Restore Bootstrap WebJar Assets

## 12:04 - Restore Bootstrap WebJar Assets

### Request

Diagnose and fully repair the production site after the user reported that some
CSS loaded while other styling and a Bootstrap URL failed with 404.

### Project Context

The authoritative `A:\Projects\christopherbell.dev` checkout was dirty, ahead,
and stale, so it was preserved untouched. Work ran in
`A:\Projects\christopherbell.dev-worktrees\bootstrap-assets-1339` on
`codex/issue-1339-bootstrap-assets` from refreshed origin/main
`2b40bd860d9e4e05aa18b4dd63e13a390d41208e`. Production is native Windows on
port 8080 and was validated first on alternate port 8091.

### Root Cause and Work Completed

Dependency update commit `20290b2f` moved Bootstrap from 5.3.3 to 5.3.8 but left
runtime templates/CSS imports and both exact security boundaries pinned to
5.3.3. Production requested missing 5.3.3 assets, while valid 5.3.8 assets were
403.

Created issue #1339 and updated 19 spoke files: all runtime WebJar references,
`SecurityConfig`, `StaticAssetRequestMatcher`, security/CSS docs, dependency-
derived JavaScript coverage, public matcher tests, and a direct
`StaticAssetRequestMatcherTest`. Commits `09606ebd87feacc9ad5828c0203fc6dd7ffd4d55`
and `facfa97cdb33dd144fbe4aedae5cbc2e45fc2ea3` were pushed. PR #1340 passed
review and CI, then squash-merged as
`5bd14e994a6130a32166602a6f272581abc53525`; issue #1339 closed automatically.

Builder artifacts include the completed spec, implementation plan, test report,
spoke update/review, work closure, and central work record for this repair.

### Decisions

Pinned runtime/security paths to the packaged 5.3.8 version and added a test that
derives the version from `website/build.gradle.kts` so a future dependency-only
bump fails. Kept the exact WebJar allowlist instead of broadening `/webjars/**`.
After independent review exposed a direct matcher-test gap, added boundary tests
for GET/POST, current/obsolete versions, and unrelated namespaces. Preserved the
authoritative dirty checkout and did not weaken protected production ACLs.

### Validation

- TDD RED reproduced 5.3.3 versus 5.3.8; GREEN contract tests passed.
- Focused `SecurityConfigTest` passed 16/16 and direct matcher coverage passed.
- `:website:jsTest` passed 312/312.
- Final `:website:check` was BUILD SUCCESSFUL in 3m20s with 1,610 Java tests,
  150 Pester tests, zero failures/errors, and 3 skipped Java tests.
- Alternate-port packaged app readiness was UP; pages and current Bootstrap CSS/
  JS returned 200 with exact signatures; obsolete paths returned 403; browser
  computed styles and logs passed.
- Independent re-review reported no actionable findings and Ready to merge: Yes.
- GitHub Linux/macOS/Windows builds, CodeQL, and dependency review passed.
- Production rotated from PID 33024 to PID 2956. A transient readiness 503 became
  200 UP; liveness was UP. Public current assets return 200, obsolete assets 403,
  login/signup reference 5.3.8, browser styles load with no logs, and
  ChristopherBellDev, MongoDB, and cloudflared are Running/Automatic.

### Current State

The feature branch is merged and issue #1339 is closed. The isolated worktree
retains only the known CRLF-only `gradlew.bat` checkout artifact, which was never
staged. The authoritative dirty checkout remains untouched. Production is
healthy on the merged release.

### Follow-ups

No issue-scoped follow-up remains. The test report records an unrelated local-
profile WFL duplicate-key catch-up log for future WFL work.
