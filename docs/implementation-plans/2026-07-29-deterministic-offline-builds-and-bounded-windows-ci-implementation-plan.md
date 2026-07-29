# Deterministic Offline Builds and Bounded Windows CI Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use `superpowers:executing-plans` task-by-task and invoke `write-jane-street-style-code` before every code edit.

**Goal:** Resolve #1302-#1305 with commit-stable artifact versions, a verified reusable sensor cache, Windows Pester in `check`, and bounded concurrency-aware CI.

**Architecture:** Root Gradle providers resolve an explicit release version or exact Git identity. Website build logic owns a checksum-first cache resolver with an injected download effect and four-path verification task. Windows-only task wiring preserves other platforms. Parsed YAML/source contract tests make the build and CI rules executable.

**Tech Stack:** Gradle 9.6.1 Kotlin DSL, Java 25, JUnit 5, AssertJ, PowerShell 7, Windows PowerShell 5.1, Pester 5.9.0, GitHub Actions.

## Document Status

ready-for-execution

## Objective

Implement and deliver trusted issues #1302-#1305 from `044299c8876dc3c421afac191194a8bcdeaa1260`, including RED/GREEN evidence, Windows testing, PR/merge, production verification, Builder reporting, and closure.

## Goals

- Same commit means same development version; explicit validated release override remains possible.
- Cold sensor download is bounded, verified, atomic, and reusable after `clean` with `--offline`.
- Windows `build` runs all three existing Pester suites with exactly Pester 5.9.0 and retains NUnit XML.
- Superseded PR CI cancels, main pushes remain independent, and jobs/critical steps are bounded.

## Inputs

Spec `docs/specs/2026-07-29-deterministic-offline-builds-and-bounded-windows-ci.md`; trusted bodies #1302-#1305 by `azurras` with no comments/attachments; inspected Gradle/workflow/test files; required Jane Street/TDD/debugging workflows.

## Branch

`codex/issues-1302-1305-20260729` at `A:\Projects\christopherbell.dev-worktrees\issues-1302-1305-20260729`, based on `origin/main` SHA `044299c8876dc3c421afac191194a8bcdeaa1260`.

## Non-Goals

Changing runtime sensor APIs, upgrading/vendoring LibreHardwareMonitor, running Pester off Windows, canceling main pushes, changing unrelated workflows, or exposing caller-controlled artifact URLs/commands.

## Assumptions

Git exists in repository builds; Gradle user home can retain a cache; GitHub Windows can install CurrentUser Pester 5.9.0; existing Pester tasks/NUnit paths are authoritative; observed five-to-six-minute builds fit 30/20-minute budgets.

## Open Questions

None. The approved spec fixes inputs, precedence, cache semantics, timeout values, task wiring, and CI budgets.

## Task Breakdown

### Task 1 - Add failing build and workflow contracts

Sequence / dependencies:
- First; establishes RED for all four issues.

Implementation notes:
- Required skill: `write-jane-street-style-code` before any code edits.
- Before-Edit Brief:
  - Behavior: tests require deterministic version/cache markers and parsed CI Pester/concurrency/timeouts.
  - Invariants: read-only, no network, existing immutable-action checks preserved.
  - Boundary/API: repository Gradle text and `.github/workflows/ci.yml` YAML nodes.
  - Effects and failures: file reads only; missing configuration yields narrow assertions.
  - Tests and evidence: run the two focused classes and capture failures before implementation.

#### Code Edit 1.1
- File: `website/src/test/java/dev/christopherbell/configuration/BuildAutomationConfigurationTest.java`
- Lines: 1-45
- Action: add

Proposed:
```java
package dev.christopherbell.configuration;

import static org.assertj.core.api.Assertions.assertThat;

import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Path;
import org.junit.jupiter.api.Test;

class BuildAutomationConfigurationTest {
  private static final Path ROOT = locateRepositoryRoot();

  @Test
  void artifactVersionUsesReleaseInputOrExactCommitWithoutClockInputs() throws IOException {
    var script = Files.readString(ROOT.resolve("build.gradle.kts"));
    assertThat(script).contains("releaseVersion", "RELEASE_VERSION",
        "0.0.0-dev.", "verifyDeterministicVersion");
    assertThat(script).doesNotContain("LocalDate", "BUILD_NUMBER");
  }

  @Test
  void sensorPreparationUsesVerifiedOfflineCacheAndBoundedDownload() throws IOException {
    var script = Files.readString(ROOT.resolve("website/build.gradle.kts"));
    assertThat(script).contains("gradleUserHomeDir", "isOffline", "connectTimeout",
        "readTimeout", "verifySensorArchiveResolution");
    assertThat(script).doesNotContain(".toURL().openStream()");
  }
}
```

Add the same bounded repository-root locator used by `GitHubAutomationConfigurationTest`.

Verification:
- `.\gradlew.bat :website:test --tests '*BuildAutomationConfigurationTest'`

#### Code Edit 1.2
- File: `website/src/test/java/dev/christopherbell/configuration/GitHubAutomationConfigurationTest.java`
- Lines: after 55
- Action: add

Proposed:
```java
@Test
void ciRunsPinnedWindowsPesterAndRetainsItsNunitResults() throws IOException {
  var workflow = readYaml(".github/workflows/ci.yml");
  var steps = workflow.at("/jobs/build/steps");
  var install = stepNamed(steps, "Install Pester 5.9.0");
  assertThat(install.path("if").asText()).isEqualTo("runner.os == 'Windows'");
  assertThat(install.path("run").asText()).contains("-RequiredVersion 5.9.0");
  assertThat(stepUsing(steps, UPLOAD_ARTIFACT).at("/with/path").asText())
      .contains("shared-folder-pester/*.xml");
}

@Test
void ciCancelsOnlySupersededPullRequestsAndBoundsWork() throws IOException {
  var workflow = readYaml(".github/workflows/ci.yml");
  assertThat(workflow.at("/concurrency/group").asText()).contains("github.workflow");
  assertThat(workflow.at("/concurrency/cancel-in-progress").asText())
      .contains("github.event_name == 'pull_request'");
  assertThat(workflow.at("/jobs/build/timeout-minutes").asInt()).isEqualTo(30);
  assertThat(workflow.at("/jobs/build/strategy/fail-fast").asBoolean()).isFalse();
}
```

Add `stepNamed(JsonNode, String)` beside `stepUsing` and assert 20-minute build, 5-minute setup/install/upload limits.

Verification:
- `.\gradlew.bat :website:test --tests '*GitHubAutomationConfigurationTest'`

### Task 2 - Make artifact versions deterministic (#1302)

Sequence / dependencies:
- After Task 1 RED; first implementation commit.

Implementation notes:
- Required skill: `write-jane-street-style-code` before any code edits.
- Before-Edit Brief:
  - Behavior: property/env release override wins; otherwise exact HEAD yields one version.
  - Invariants: no date/run input, full lowercase SHA, bounded safe release syntax.
  - Boundary/API: Gradle `project.version` and Spring Boot build-info remain consumers.
  - Effects and failures: one `git rev-parse`; invalid release/identity fails configuration clearly.
  - Tests and evidence: source test RED; verification task and differing BUILD_NUMBER outputs GREEN.

#### Code Edit 2.1
- File: `build.gradle.kts`
- Lines: 1-13
- Action: replace

Current:
```kotlin
import java.time.LocalDate
import java.time.format.DateTimeFormatter

plugins {
    id("org.springframework.boot") version "4.1.0" apply false
    id("io.spring.dependency-management") version "1.1.7" apply false
    java
}

group = "dev.christopherbell"
val buildNumber = System.getenv("BUILD_NUMBER") ?: "0"
val dateVersion = LocalDate.now().format(DateTimeFormatter.ofPattern("yyyy.MM.dd"))
version = "$dateVersion.$buildNumber"
```

Proposed:
```kotlin
plugins {
    id("org.springframework.boot") version "4.1.0" apply false
    id("io.spring.dependency-management") version "1.1.7" apply false
    java
}

fun validatedReleaseVersion(raw: String): String {
    val value = raw.trim()
    if (raw != value || !value.matches(Regex("[0-9A-Za-z][0-9A-Za-z._+-]{0,127}"))) {
        throw GradleException("releaseVersion must be 1-128 safe version characters.")
    }
    return value
}

fun developmentVersion(commit: String): String {
    val normalized = commit.trim().lowercase()
    if (!normalized.matches(Regex("[0-9a-f]{40}"))) {
        throw GradleException("Git HEAD must resolve to a full 40-character commit SHA.")
    }
    return "0.0.0-dev.$normalized"
}

val sourceGitCommit = providers.exec {
    commandLine("git", "rev-parse", "HEAD")
    workingDir(rootProject.projectDir)
}.standardOutput.asText.map(String::trim)
val explicitReleaseVersion = providers.gradleProperty("releaseVersion")
    .orElse(providers.environmentVariable("RELEASE_VERSION"))
group = "dev.christopherbell"
version = explicitReleaseVersion.map(::validatedReleaseVersion)
    .orElse(sourceGitCommit.map(::developmentVersion)).get()

tasks.register("verifyDeterministicVersion") {
    doLast {
        val commit = sourceGitCommit.get()
        check(developmentVersion(commit) == developmentVersion(commit))
        if (!explicitReleaseVersion.isPresent) check(version == developmentVersion(commit))
    }
}
tasks.named("check") { dependsOn("verifyDeterministicVersion") }
```

Verification:
- Compare `properties --property version` under `BUILD_NUMBER=1` and `BUILD_NUMBER=999`.
- `.\gradlew.bat verifyDeterministicVersion`

### Task 3 - Add verified durable sensor cache (#1303)

Sequence / dependencies:
- After Task 2 commit; preserve an isolated review unit.

Implementation notes:
- Required skill: `write-jane-street-style-code` before any code edits.
- Before-Edit Brief:
  - Behavior: valid cache avoids network; cold fetch is bounded/atomic; offline missing/corrupt/unavailable fail closed.
  - Invariants: pinned URI/digests, no partial publication, every member verified, cache survives `clean`.
  - Boundary/API: existing task/resource/JAR paths remain externally stable.
  - Effects and failures: cache read, optional HTTPS GET, partial rename, extraction; distinct actionable failures.
  - Tests and evidence: structural RED; four injected-effect paths plus real online/cache/offline GREEN.

#### Code Edit 3.1
- File: `website/build.gradle.kts`
- Lines: 78-134
- Action: replace

Current:
```kotlin
val libreHardwareMonitorUri = URI(
    "https://github.com/LibreHardwareMonitor/LibreHardwareMonitor/releases/download/v0.9.6/LibreHardwareMonitor.zip")
val sensorResourceDirectory = layout.buildDirectory.dir("generated/sensor-resources")
val prepareSensorResources by tasks.registering {
    val archive = layout.buildDirectory.file("sensor-downloads/LibreHardwareMonitor-0.9.6.zip")
    outputs.dir(sensorResourceDirectory)
    doLast {
        val archivePath = archive.get().asFile.toPath()
        Files.createDirectories(archivePath.parent)
        libreHardwareMonitorUri.toURL().openStream().use { input ->
            Files.copy(input, archivePath, StandardCopyOption.REPLACE_EXISTING)
        }
        if (sha256(archivePath) != libreHardwareMonitorArchiveSha256) {
            Files.deleteIfExists(archivePath)
            throw GradleException("LibreHardwareMonitor archive SHA-256 verification failed.")
        }
    }
}
```

Proposed:
```kotlin
val sensorConnectTimeout = Duration.ofSeconds(10)
val sensorReadTimeout = Duration.ofSeconds(30)
val sensorArchiveCache = providers.gradleProperty("sensorArchiveCache")
    .map(::file)
    .orElse(provider { File(gradle.gradleUserHomeDir,
        "caches/christopherbell.dev/sensors/LibreHardwareMonitor-0.9.6.zip") })

fun resolvePinnedArchive(
    cache: Path, expected: String, offline: Boolean, downloadTo: (Path) -> Unit
): Path {
    if (Files.exists(cache)) {
        if (sha256(cache) != expected) throw GradleException("Cached sensor archive checksum failed: $cache")
        return cache
    }
    if (offline) throw GradleException("Sensor archive is not cached; prepare it online once.")
    val partial = cache.resolveSibling("${cache.fileName}.part")
    Files.createDirectories(cache.parent)
    try {
        downloadTo(partial)
        if (sha256(partial) != expected) throw GradleException("Downloaded sensor archive checksum failed.")
        moveAtomicallyOrReplace(partial, cache)
        return cache
    } catch (failure: Exception) {
        Files.deleteIfExists(partial)
        if (failure is GradleException) throw failure
        throw GradleException("Sensor upstream was unavailable within configured timeouts.", failure)
    }
}

fun downloadPinnedArchive(uri: URI, target: Path) {
    val connection = uri.toURL().openConnection()
    connection.connectTimeout = sensorConnectTimeout.toMillis().toInt()
    connection.readTimeout = sensorReadTimeout.toMillis().toInt()
    // Require HTTP 2xx, copy, close, and disconnect.
}

val prepareSensorResources by tasks.registering {
    inputs.property("archiveUri", libreHardwareMonitorUri.toString())
    inputs.property("archiveSha256", libreHardwareMonitorArchiveSha256)
    inputs.property("offline", gradle.startParameter.isOffline)
    outputs.file(sensorArchiveCache)
    outputs.dir(sensorResourceDirectory)
    outputs.upToDateWhen { false }
    doLast {
        val archive = resolvePinnedArchive(sensorArchiveCache.get().toPath(),
            libreHardwareMonitorArchiveSha256, gradle.startParameter.isOffline) {
            downloadPinnedArchive(libreHardwareMonitorUri, it)
        }
        // Extract allowlisted members and verify every pinned SHA-256.
    }
}
```

Add `verifySensorArchiveResolution`: use task-temporary byte fixtures/injected writers to prove cold-online, cached-offline/no writer, corrupt checksum/no publish, and unavailable writer/no partial. Attach it to `check`. Atomic-move fallback catches only `AtomicMoveNotSupportedException`.

Verification:
- `.\gradlew.bat :website:verifySensorArchiveResolution`
- Cold `.\gradlew.bat :website:prepareSensorResources -PsensorArchiveCache=<empty>`.
- `.\gradlew.bat clean :website:processResources --offline -PsensorArchiveCache=<same>`.

### Task 4 - Wire pinned Pester and bounded CI (#1304, #1305)

Sequence / dependencies:
- After Task 3; final code-changing unit.

Implementation notes:
- Required skill: `write-jane-street-style-code` before any code edits.
- Before-Edit Brief:
  - Behavior: Windows `build` runs three Pester suites; superseded PR work cancels; all platform jobs stay bounded.
  - Invariants: exact Pester 5.9.0, NUnit retained, non-Windows no Windows tools, main pushes not canceled, actions SHA-pinned.
  - Boundary/API: Gradle `check`, CI workflow, existing task/report names.
  - Effects and failures: module install/test processes; timeouts and failures surface through jobs/artifacts.
  - Tests and evidence: parsed YAML RED/GREEN and local dual-shell Pester XML.

#### Code Edit 4.1
- File: `website/build.gradle.kts`
- Lines: 350-363
- Action: replace

Current:
```kotlin
tasks.register("sharedFolderVerification") {
    group = LifecycleBasePlugin.VERIFICATION_GROUP
    description = "Runs shared-folder Java, browser, worker, and operations regression coverage."
    dependsOn(
        tasks.named("test"),
        tasks.named("jsTest"),
        sharedFolderWorkerPester,
        sharedFolderOperationsPwshPester,
        sharedFolderOperationsWindowsPowerShellPester)
}

tasks.named("check") {
    dependsOn("jsTest")
}
```

Proposed:
```kotlin
val windowsPesterVerification = listOf(
    sharedFolderWorkerPester,
    sharedFolderOperationsPwshPester,
    sharedFolderOperationsWindowsPowerShellPester)

tasks.register("sharedFolderVerification") {
    group = LifecycleBasePlugin.VERIFICATION_GROUP
    description = "Runs shared-folder Java, browser, worker, and operations regression coverage."
    dependsOn(tasks.named("test"), tasks.named("jsTest"), windowsPesterVerification)
}

tasks.named("check") {
    dependsOn("jsTest")
    if (System.getProperty("os.name").startsWith("Windows", ignoreCase = true)) {
        dependsOn(windowsPesterVerification)
    }
}
```

Verification:
- `.\gradlew.bat :website:sharedFolderWorkerPester :website:sharedFolderOperationsPwshPester :website:sharedFolderOperationsWindowsPowerShellPester`
- Inspect three NUnit XML files for zero failures.

#### Code Edit 4.2
- File: `.github/workflows/ci.yml`
- Lines: 1-55
- Action: replace

Current:
```yaml
name: CI Build

permissions:
  contents: read

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        java-version: [ 25 ]
        os: [ ubuntu-latest, macos-latest, windows-latest ]
```

Proposed:
```yaml
name: CI Build

permissions:
  contents: read

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

concurrency:
  group: "${{ github.workflow }}-${{ github.event.pull_request.number || github.ref }}"
  cancel-in-progress: "${{ github.event_name == 'pull_request' }}"

jobs:
  build:
    runs-on: ${{ matrix.os }}
    timeout-minutes: 30
    strategy:
      fail-fast: false
      matrix:
        java-version: [ 25 ]
        os: [ ubuntu-latest, macos-latest, windows-latest ]
```

Give checkout/JDK/Node/Gradle 5 minutes; add Windows-only `Install Pester 5.9.0` with pinned CurrentUser install/import assertion and 5 minutes; give both builds 20 minutes; give failure upload 5 minutes and include `**/build/test-results/shared-folder-pester/*.xml`.

Verification:
- `.\gradlew.bat :website:test --tests '*GitHubAutomationConfigurationTest'`
- `git diff --check`

### Task 5 - Full delivery

Sequence / dependencies:
- After Tasks 2-4 commits and focused GREEN.

Implementation notes:
- No code edit. Rebase on latest `origin/main`, rerun exact-publish evidence, preserve dirty authoritative checkout and wrapper artifact.

Verification:
- `.\gradlew.bat build --console=plain`
- Exact Java/JavaScript/Pester XML totals.
- Packaged alternate-port runtime smoke.
- `gh pr checks <number> --watch`; merged-main CI/CodeQL; exact-SHA production health/services.

## Code Changes

- `BuildAutomationConfigurationTest.java`: add Gradle contract tests (1.1).
- `GitHubAutomationConfigurationTest.java`: add CI contract tests/helper (1.2).
- `build.gradle.kts`: replace date/run versioning; add verification (2.1).
- `website/build.gradle.kts`: cache/download/verification and Windows Pester wiring (3.1, 4.1).
- `.github/workflows/ci.yml`: concurrency, Pester, and timeouts (4.2).

## Files and Modules

Builder artifacts; root/module Gradle; CI YAML; configuration tests; existing Windows Pester modules/tests; unchanged application runtime code.

## Unit Testing

RED/GREEN focused Java configuration classes; `verifyDeterministicVersion`; four-path `verifySensorArchiveResolution`; existing automation tests.

## Local Testing

Compare versions under two BUILD_NUMBER values plus valid/invalid release override; cold online and clean cached-offline sensor generation; corrupt/unavailable failure paths; all three dual-shell Pester tasks/NUnit; full isolated build; packaged app on non-8080 with isolated Mongo and root/liveness/readiness/static-resource requests.

## Validation

All issue/spec criteria, focused/full local tests, strict verification, Pester XML, GitHub matrix/dependency review/CodeQL, mainline checks, exact-SHA production routes/services, and Builder artifacts pass.

## Rollback or Recovery

Separate concern commits allow targeted revert. Production retains automatic previous-release rollback. A corrupt cache is recovered only by deleting the exact reported repository-specific cache file then rerunning online; checksums are never bypassed.

## Risks

Cold upstream availability (timeouts/warm cache); Kotlin DSL/provider interactions (focused/full tasks); dual-shell module discovery (pinned install/local XML); timeout sizing (large observed margin); concurrency expression syntax (parsed YAML/live PR); mainline movement (rebase/retest).

## Completion Criteria

#1302-#1305 are merged, all local/CI/runtime/production evidence is green, Builder report/memory is pushed, and every issue closes without unresolved gaps.
