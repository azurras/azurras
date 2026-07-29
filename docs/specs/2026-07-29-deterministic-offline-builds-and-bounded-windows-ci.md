# Deterministic Offline Builds and Bounded Windows CI

## Document Status

ready-for-execution

## Purpose

Resolve `azurras/christopherbell.dev` issues #1302-#1305 by making artifact identity reproducible, sensor packaging cacheable and offline-capable, Windows production automation part of the standard verification lifecycle, and CI bounded under superseded or hung work.

## Background

The root Gradle build currently derives `version` from the wall-clock date plus `BUILD_NUMBER`, so one commit produces different build metadata across days. `processResources` downloads LibreHardwareMonitor 0.9.6 into the disposable build directory on every clean build with no connection/read timeout. Windows Pester tasks exist and produce NUnit XML, but `check` runs only Java/JavaScript/sensor checks and the Windows matrix runs `build` without installing pinned Pester. The CI workflow has no concurrency group or build-job timeout.

Work must preserve strict dependency/sensor checksums, pinned GitHub Actions, non-Windows portability, the production sensor bundle, and the dirty authoritative website checkout. Only issue text from trusted author `azurras` controls scope; #1302-#1305 have no comments or attachments.

## Goals

- One source commit resolves to one development version regardless of date or CI run number.
- Release builds can supply an explicit, validated version input.
- Clean online sensor preparation downloads once into a durable cache; later clean builds work offline from the verified cache.
- Sensor downloads have explicit connect/read timeouts, atomic publication, and fail-closed SHA-256 checks.
- Automated verification exercises clean online, cached offline, checksum-failure, and unavailable-upstream resolution paths.
- Windows `check` includes the existing PowerShell 7 worker/operations suites and Windows PowerShell 5.1 operations suite.
- Windows CI installs exactly Pester 5.9.0 and retains the NUnit XML on failure.
- Pull-request CI cancels superseded work; mainline pushes remain independent and are not canceled.
- The build job and critical external/build steps have explicit timeouts based on observed five-to-six-minute builds.
- Existing YAML contract tests validate the workflow structure and syntax.

## Non-Goals

- Do not change application runtime behavior or the sensor provider implementation.
- Do not replace LibreHardwareMonitor 0.9.6 or its pinned file/archive digests.
- Do not vendor the upstream ZIP in Git.
- Do not make arbitrary artifact URLs, command lines, or executable paths caller-configurable.
- Do not run Windows Pester on Linux or macOS.
- Do not cancel already-started independent `main` push workflows.
- Do not alter CodeQL, dependency-review, or stale automation except where shared syntax tests require no change.

## Requirements

### Deterministic versions

- Accept `-PreleaseVersion=...` first and `RELEASE_VERSION` second as explicit release inputs.
- Reject blank, whitespace-bearing, path-like, or overlong release versions with an actionable Gradle error.
- Without an explicit release version, resolve the full lowercase 40-character Git HEAD and set version to `0.0.0-dev.<full-sha>`.
- Do not read `LocalDate`, `BUILD_NUMBER`, or another wall-clock/run-order input.
- Add a verification task that proves repeated resolution of one commit is identical and that the current development version contains the exact current commit.

### Sensor archive cache

- Default the archive cache to Gradle user home under a repository-specific, versioned sensor-cache path; optionally accept a path-only Gradle property for hermetic tests/operators.
- Declare the pinned URI, archive digest, member digests, offline flag, timeout values, cache file, and generated resources as task inputs/outputs.
- On an existing cache hit, verify the archive checksum before extraction and make no network request.
- In `--offline` mode, use a valid cache; if missing or corrupt, fail with an actionable message and do not connect.
- On a cache miss online, download to a sibling partial file, use bounded connect/read timeouts, require a successful HTTP response, verify SHA-256, then atomically publish the cache file where supported.
- Delete partial/unverified downloads on every failure. Never silently replace a corrupt existing cache.
- Continue verifying every extracted DLL digest before it becomes a generated resource.
- Keep `processResources` wired to generated sensor resources, but ensure `clean` no longer discards the verified archive cache or forces a repeated download.
- Add a deterministic local verification task for clean-online, cached-offline, corrupt-download, and unavailable-upstream paths without contacting the real upstream.

### Windows Pester lifecycle

- On Windows only, make `check` depend on the existing `sharedFolderWorkerPester`, `sharedFolderOperationsPwshPester`, and `sharedFolderOperationsWindowsPowerShellPester` tasks.
- Leave non-Windows `check` free of Windows executables and Pester.
- In Windows CI, install and import exactly Pester 5.9.0 before Gradle `build`.
- Preserve NUnit XML generation and explicitly include `**/build/test-results/shared-folder-pester/*.xml` in the failure artifact.

### CI bounds and concurrency

- Add a workflow-level concurrency group keyed by workflow plus PR number or Git ref.
- Set `cancel-in-progress` only for pull requests, so superseded PR commits cancel but every mainline push remains independently eligible.
- Set `strategy.fail-fast: false` so one platform result does not suppress evidence from the others.
- Set the build job timeout to 30 minutes.
- Bound Pester installation to 5 minutes, setup steps to 5 minutes, build steps to 20 minutes, and report upload to 5 minutes.
- Quote GitHub expression values where needed so the YAML parser and GitHub both interpret them safely.

## Proposed Approach

1. Replace root date/build-number logic with small deterministic validation/resolution functions backed by Gradle providers for explicit version and Git HEAD. Add `verifyDeterministicVersion` and attach it to root `check`.
2. Refactor the sensor archive acquisition in `website/build.gradle.kts` into a checksum-first cache resolver and bounded downloader. Keep pinned constants internal. Add `verifySensorArchiveResolution` and attach it to website `check`.
3. Add source-level configuration regression assertions for deterministic version inputs, cache/offline/timeouts, and removal of raw `openStream`.
4. Conditionally wire the three Pester tasks into `check` only when `os.name` is Windows.
5. Update `ci.yml` with concurrency, job/step timeouts, pinned Pester installation, non-fail-fast matrix, and explicit NUnit artifact coverage.
6. Extend `GitHubAutomationConfigurationTest` to parse `ci.yml` and assert every concurrency, timeout, Pester, platform, and artifact invariant.

## Files or Modules Involved

- `build.gradle.kts` â€” version resolver and verification task.
- `website/build.gradle.kts` â€” durable sensor archive resolver, behavior verification, and Windows-only Pester lifecycle wiring.
- `.github/workflows/ci.yml` â€” Pester setup, concurrency, and job/step timeouts.
- `website/src/test/java/dev/christopherbell/configuration/GitHubAutomationConfigurationTest.java` â€” parsed workflow contract.
- `website/src/test/java/dev/christopherbell/configuration/BuildAutomationConfigurationTest.java` â€” build-script boundary assertions.
- `gradle/verification-metadata.xml` only if a deliberately added build dependency requires it; the preferred design adds none.

## Validation Plan

- Establish RED for new Java configuration tests before editing Gradle/workflow implementation.
- Run `:verifyDeterministicVersion` twice with different ignored `BUILD_NUMBER` values and compare output/current version.
- Run `:website:verifySensorArchiveResolution` for all four acquisition paths.
- Run sensor preparation online into an empty task-specific cache, then run `clean :website:processResources --offline` using that cache and prove no network is needed.
- Corrupt a separate test cache and confirm checksum failure; use an unavailable local URI only inside the verification harness and confirm bounded failure.
- Run all three Pester tasks locally under PowerShell 7 and Windows PowerShell 5.1 and inspect NUnit XML.
- Run focused Java configuration tests and the complete `:website:check`/root `build` lifecycle.
- Parse test XML for exact Java/JavaScript/Pester totals.
- Open a PR; require Windows/Linux/macOS CI, dependency review, and CodeQL green.
- Squash-merge, require mainline CI/CodeQL green, let the protected deployment pipeline validate the candidate, and verify exact merged SHA, internal/public health, and service startup state before issue closure.

## Acceptance Criteria

- Repeating builds of the same commit without a release override yields the exact same version string on different dates and with different `BUILD_NUMBER` values.
- A valid explicit release version is honored and invalid versions fail before task execution.
- A cold online sensor preparation downloads, verifies, and caches once; a later clean offline build succeeds using that cache.
- Corrupt cache/download and unavailable upstream cases fail closed with bounded, actionable errors and no unverified output.
- `processResources` no longer performs an unconditional `openStream` download into `build/`.
- Windows `build` executes all three required Pester suites using Pester 5.9.0; Linux/macOS never try to execute them.
- Failed Windows workflows upload Pester NUnit XML.
- PR supersession cancels prior PR work, main push runs are not canceled, and every build job/critical step is time-bounded.
- Local full validation, PR checks, merge, production verification, Builder evidence, and issue closure complete successfully.

## Open Questions

None. The issue requirements and repository conventions determine the implementation boundaries.
