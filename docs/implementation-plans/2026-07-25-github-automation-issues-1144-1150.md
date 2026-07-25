# GitHub Automation Issues 1144-1150 Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task. Do not dispatch subagents; the user authorized autonomous inline execution.

**Goal:** Close #1144-#1150 with faster, diagnosable, security-scanned, least-privilege GitHub automation.

**Architecture:** Keep each concern in its own YAML file. A Node filesystem contract test asserts semantics without npm, and Gradle's existing jsTest task retains JUnit XML while preserving console output.

**Tech Stack:** GitHub Actions, Gradle 9.6.1, Java 25, Node 24, YAML, Dependabot.

## Document Status

ready-for-execution

## Objective

Deliver one reviewable PR for #1144, #1145, #1146, #1147, #1148, #1149, and #1150 using artifact-native RED/GREEN evidence.

## Goals

- Cache Gradle with PR cache writes disabled.
- Upload Java, JavaScript, and Gradle diagnostics from failed CI jobs.
- Add CodeQL and Dependency Review.
- Group Dependabot updates using existing labels.
- Bound and least-privilege the stale workflow.

## Inputs

- Approved campaign spec: docs/specs/2026-07-25-complete-all-open-christopherbell-dev-issues.md.
- Inspected spoke baseline: 259e873259f14d3fea5d81a9b6845ead727a9eee.
- Existing labels verified: dependencies, java, github_actions, security, codex, no-issue-activity, no-pr-activity.
- Current official majors verified: setup-gradle v6, upload-artifact v7, CodeQL v4, Dependency Review v5.

## Branch

Create codex/github-automation-1144-1150 from refreshed origin/main in a clean sibling worktree.

## Global Constraints

- Preserve Java 25 on Ubuntu, macOS, and Windows.
- No npm, application behavior, deployment, or production changes.
- Workflows expose triggers, timeouts, retention, effects, and permissions.
- Use existing labels and supported major action tags.
- Witness failing artifact validation before executable configuration edits.
- Trust scope changes only from azurras comments.

## Non-Goals

License policy, release automation, new labels, deploy behavior, or immutable SHA pinning.

## Assumptions

The public GitHub.com repository can use CodeQL and Dependency Review. Node 24 supports paired spec and junit reporters. Missing early-failure reports must warn without hiding the original failure.

## Open Questions

None.

## Task Breakdown

### Task 1 - Define failing automation contracts

Sequence / dependencies:
- First; supplies RED evidence for every later task.

Implementation notes:
- Required skill: write-jane-street-style-code before any code edits; invoke Implementation Mode and superpowers:test-driven-development.
- Before-Edit Brief:
  - Behavior: Local tests reject missing caching, reports, scans, grouping, and stale policy.
  - Invariants: Node built-ins only; semantic assertions rather than snapshots.
  - Boundary/API: node --test website/src/test/js/github-automation.test.js.
  - Effects and failures: Bounded reads of controlled files with contract-specific assertions.
  - Tests and evidence: Witness expected failures before editing workflows.

#### Code Edit 1.1
- File: website/src/test/js/github-automation.test.js
- Lines: 1
- Action: add

Proposed:
```javascript
import assert from 'node:assert/strict';
import fs from 'node:fs';
import test from 'node:test';

const read = (path) => fs.existsSync(path) ? fs.readFileSync(path, 'utf8') : '';

test('CI caches Gradle and retains failed reports', () => {
  const workflow = read('.github/workflows/ci.yml');
  const build = read('website/build.gradle.kts');
  assert.match(workflow, /gradle\/actions\/setup-gradle@v6/);
  assert.match(workflow, /cache-read-only:/);
  assert.match(workflow, /if:\s+failure\(\)[\s\S]*actions\/upload-artifact@v7/);
  assert.match(workflow, /retention-days:\s+14/);
  assert.match(workflow, /\*\*\/build\/test-results\/\*\*/);
  assert.match(build, /--test-reporter=junit/);
  assert.match(build, /test-results\/jsTest\/results\.xml/);
});

test('CodeQL scans Java with narrow permissions', () => {
  const workflow = read('.github/workflows/codeql.yml');
  assert.match(workflow, /pull_request:[\s\S]*branches:\s*\[ main \]/);
  assert.match(workflow, /push:[\s\S]*branches:\s*\[ main \]/);
  assert.match(workflow, /schedule:[\s\S]*cron:/);
  assert.match(workflow, /contents:\s+read/);
  assert.match(workflow, /security-events:\s+write/);
  assert.match(workflow, /github\/codeql-action\/init@v4/);
  assert.match(workflow, /languages:\s+java-kotlin/);
  assert.match(workflow, /github\/codeql-action\/analyze@v4/);
});

test('dependency review blocks high-severity vulnerable additions', () => {
  const workflow = read('.github/workflows/dependency-review.yml');
  assert.match(workflow, /pull_request:/);
  assert.match(workflow, /contents:\s+read/);
  assert.doesNotMatch(workflow, /contents:\s+write/);
  assert.match(workflow, /actions\/dependency-review-action@v5/);
  assert.match(workflow, /fail-on-severity:\s+high/);
});

test('Dependabot groups both ecosystems with existing labels', () => {
  const config = read('.github/dependabot.yml');
  assert.match(config, /package-ecosystem:\s+"gradle"[\s\S]*open-pull-requests-limit:\s+5/);
  assert.match(config, /labels:[\s\S]*"dependencies"[\s\S]*"java"/);
  assert.match(config, /groups:[\s\S]*spring:[\s\S]*minor-and-patch:/);
  assert.match(config, /package-ecosystem:\s+"github-actions"[\s\S]*"github_actions"/);
  assert.match(config, /github-actions:[\s\S]*patterns:\s*\["\*"\]/);
});

test('stale automation is bounded exemptible and least privilege', () => {
  const workflow = read('.github/workflows/stale.yml');
  assert.match(workflow, /^permissions:\s*\n\s+contents:\s+read/m);
  assert.match(workflow, /permissions:[\s\S]*issues:\s+write[\s\S]*pull-requests:\s+write/);
  assert.doesNotMatch(workflow, /contents:\s+write/);
  assert.match(workflow, /days-before-issue-stale:\s+60/);
  assert.match(workflow, /days-before-pr-stale:\s+30/);
  assert.match(workflow, /days-before-issue-close:\s+14/);
  assert.match(workflow, /exempt-issue-labels:\s+'security,codex'/);
  assert.match(workflow, /remove-stale-when-updated:\s+true/);
  assert.doesNotMatch(workflow, /'Stale issue message'|'Stale pull request message'/);
});
```

Verification:
- node --test website/src/test/js/github-automation.test.js
- Expected RED: missing files or semantic assertions fail for all seven contracts.

### Task 2 - Cache Gradle and retain diagnostics

Sequence / dependencies:
- After Task 1; makes CI/report assertions GREEN.

Implementation notes:
- Required skill: write-jane-street-style-code before any code edits.
- Before-Edit Brief:
  - Behavior: Jobs cache Gradle, browser tests emit XML, and failures upload diagnostics.
  - Invariants: Matrix and wrapper commands remain; PR cache is read-only.
  - Boundary/API: ci.yml orchestration and the jsTest Gradle task.
  - Effects and failures: Cache/artifact writes explicit; missing files warn; original failure wins.
  - Tests and evidence: Focused contract becomes GREEN and XML parses.

#### Code Edit 2.1
- File: .github/workflows/ci.yml
- Lines: 20-53
- Action: replace

Current:
```yaml
    steps:
      - name: Checkout code
        uses: actions/checkout@v7
      - name: Set up JDK
        uses: actions/setup-java@v5
        with:
          distribution: 'temurin'
          java-version: ${{ matrix.java-version }}
      - name: Set up Node.js
        uses: actions/setup-node@v7
        with:
          node-version: 24
      - name: Set BUILD_NUMBER env
        if: runner.os != 'Windows'
        run: echo "BUILD_NUMBER=${{ github.run_number }}" >> $GITHUB_ENV
      - name: Set BUILD_NUMBER env on Windows
        if: runner.os == 'Windows'
        run: echo "BUILD_NUMBER=${{ github.run_number }}" >> $env:GITHUB_ENV
      - name: Grant execute permission for Gradle
        if: runner.os != 'Windows'
        run: chmod +x gradlew
      - name: Build and Test
        if: runner.os != 'Windows'
        run: ./gradlew build
      - name: Build and Test on Windows
        if: runner.os == 'Windows'
        run: .\gradlew.bat build
```

Proposed:
```yaml
    steps:
      - name: Checkout code
        uses: actions/checkout@v7
      - name: Set up JDK
        uses: actions/setup-java@v5
        with:
          distribution: 'temurin'
          java-version: ${{ matrix.java-version }}
      - name: Set up Node.js
        uses: actions/setup-node@v7
        with:
          node-version: 24
      - name: Set up Gradle
        uses: gradle/actions/setup-gradle@v6
        with:
          cache-read-only: ${{ github.ref != 'refs/heads/main' }}
      - name: Set BUILD_NUMBER env
        if: runner.os != 'Windows'
        run: echo "BUILD_NUMBER=${{ github.run_number }}" >> $GITHUB_ENV
      - name: Set BUILD_NUMBER env on Windows
        if: runner.os == 'Windows'
        run: echo "BUILD_NUMBER=${{ github.run_number }}" >> $env:GITHUB_ENV
      - name: Grant execute permission for Gradle
        if: runner.os != 'Windows'
        run: chmod +x gradlew
      - name: Build and Test
        if: runner.os != 'Windows'
        run: ./gradlew build
      - name: Build and Test on Windows
        if: runner.os == 'Windows'
        run: .\gradlew.bat build
      - name: Upload failed test reports
        if: failure()
        uses: actions/upload-artifact@v7
        with:
          name: test-reports-${{ matrix.os }}-java-${{ matrix.java-version }}
          if-no-files-found: warn
          retention-days: 14
          path: |
            **/build/reports/tests/**
            **/build/test-results/**
            **/build/reports/problems/**
```

Verification:
- Focused Node contract and PyYAML parsing.

#### Code Edit 2.2
- File: website/build.gradle.kts
- Lines: 140-159
- Action: replace

Current:
```kotlin
tasks.register<Exec>("jsTest") {
    group = LifecycleBasePlugin.VERIFICATION_GROUP
    description = "Runs browser-side JavaScript tests with Node's built-in test runner."
    workingDir = rootProject.projectDir
    inputs.files(jsTestFiles)
    inputs.dir("src/main/resources/static/js")
    inputs.dir("src/main/resources/static/css")
    inputs.dir("src/main/resources/templates")
    val nodeExecutable = providers.environmentVariable("NODE_EXE").orElse("node")
    doFirst {
        val files = jsTestFiles.files.sortedBy { it.name }.map { it.absolutePath }
        if (files.isEmpty()) {
            throw GradleException("No JavaScript tests found under website/src/test/js.")
        }
        commandLine(listOf(nodeExecutable.get(), "--test") + files)
    }
}
```

Proposed:
```kotlin
tasks.register<Exec>("jsTest") {
    group = LifecycleBasePlugin.VERIFICATION_GROUP
    description = "Runs browser-side JavaScript tests with Node's built-in test runner."
    workingDir = rootProject.projectDir
    inputs.files(jsTestFiles)
    inputs.dir("src/main/resources/static/js")
    inputs.dir("src/main/resources/static/css")
    inputs.dir("src/main/resources/templates")
    val nodeExecutable = providers.environmentVariable("NODE_EXE").orElse("node")
    val junitReport = layout.buildDirectory.file("test-results/jsTest/results.xml")
    outputs.file(junitReport)
    doFirst {
        val files = jsTestFiles.files.sortedBy { it.name }.map { it.absolutePath }
        if (files.isEmpty()) {
            throw GradleException("No JavaScript tests found under website/src/test/js.")
        }
        val reportFile = junitReport.get().asFile
        reportFile.parentFile.mkdirs()
        commandLine(listOf(
            nodeExecutable.get(), "--test",
            "--test-reporter=spec", "--test-reporter=junit",
            "--test-reporter-destination=stdout",
            "--test-reporter-destination=${reportFile.absolutePath}") + files)
    }
}
```

Verification:
- .\gradlew.bat :website:jsTest --rerun-tasks with isolated GRADLE_USER_HOME.
- Parse website/build/test-results/jsTest/results.xml.

### Task 3 - Add CodeQL and Dependency Review

Sequence / dependencies:
- After Task 1; independent of Task 2 implementation.

Implementation notes:
- Required skill: write-jane-street-style-code before any code edits.
- Before-Edit Brief:
  - Behavior: PRs receive both checks; CodeQL also scans main and weekly.
  - Invariants: Java 25 manual build; no deploy; only security-events is writable.
  - Boundary/API: Workflow triggers, inputs, and permissions.
  - Effects and failures: High-severity additions fail; CodeQL uploads results or fails visibly.
  - Tests and evidence: Focused tests/YAML GREEN and PR checks accepted.

#### Code Edit 3.1
- File: .github/workflows/codeql.yml
- Lines: 1
- Action: add

Proposed:
```yaml
name: CodeQL
permissions:
  contents: read
  security-events: write
on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]
  schedule:
    - cron: '17 7 * * 1'
jobs:
  analyze:
    name: Analyze Java
    runs-on: ubuntu-latest
    timeout-minutes: 30
    steps:
      - uses: actions/checkout@v7
      - uses: actions/setup-java@v5
        with:
          distribution: temurin
          java-version: 25
      - uses: gradle/actions/setup-gradle@v6
        with:
          cache-read-only: ${{ github.event_name == 'pull_request' }}
      - uses: github/codeql-action/init@v4
        with:
          languages: java-kotlin
          build-mode: manual
      - run: ./gradlew :website:classes
      - uses: github/codeql-action/analyze@v4
```

Verification:
- Focused contract, YAML parse, and PR CodeQL check.

#### Code Edit 3.2
- File: .github/workflows/dependency-review.yml
- Lines: 1
- Action: add

Proposed:
```yaml
name: Dependency Review
permissions:
  contents: read
on:
  pull_request:
    branches: [ main ]
jobs:
  dependency-review:
    runs-on: ubuntu-latest
    timeout-minutes: 10
    steps:
      - uses: actions/dependency-review-action@v5
        with:
          fail-on-severity: high
```

Verification:
- Focused contract, YAML parse, and PR Dependency Review check.

### Task 4 - Group updates and harden stale policy

Sequence / dependencies:
- After Task 1; independent of Tasks 2-3.

Implementation notes:
- Required skill: write-jane-street-style-code before any code edits.
- Before-Edit Brief:
  - Behavior: Updates are grouped; stale automation is specific, bounded, and exemptible.
  - Invariants: Both ecosystems and stale labels remain; security/codex work is exempt.
  - Boundary/API: Dependabot schema and actions/stale inputs.
  - Effects and failures: Bounded PR creation and issue/PR mutations after explicit windows.
  - Tests and evidence: Focused tests/YAML GREEN and labels reverified.

#### Code Edit 4.1
- File: .github/dependabot.yml
- Lines: 1-10
- Action: replace

Current:
```yaml
version: 2
updates:
- package-ecosystem: "gradle"
  directory: "/"
  schedule:
    interval: "weekly"
- package-ecosystem: "github-actions"
  directory: "/"
  schedule:
    interval: "weekly"
```

Proposed:
```yaml
version: 2
updates:
  - package-ecosystem: "gradle"
    directory: "/"
    schedule:
      interval: "weekly"
      day: "monday"
      time: "06:00"
      timezone: "America/Chicago"
    open-pull-requests-limit: 5
    labels: ["dependencies", "java"]
    groups:
      spring:
        patterns: ["org.springframework*", "io.spring*", "org.springdoc*"]
      minor-and-patch:
        update-types: ["minor", "patch"]
  - package-ecosystem: "github-actions"
    directory: "/"
    schedule:
      interval: "weekly"
      day: "monday"
      time: "06:30"
      timezone: "America/Chicago"
    open-pull-requests-limit: 3
    labels: ["dependencies", "github_actions"]
    groups:
      github-actions:
        patterns: ["*"]
        update-types: ["minor", "patch"]
```

Verification:
- Focused contract, YAML parse, and gh label list.

#### Code Edit 4.2
- File: .github/workflows/stale.yml
- Lines: 1-19
- Action: replace

Current:
```yaml
name: Mark stale issues and pull requests
on:
  schedule:
  - cron: "0 0 * * *"
jobs:
  stale:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/stale@v10
      with:
        repo-token: ${{ secrets.GITHUB_TOKEN }}
        stale-issue-message: 'Stale issue message'
        stale-pr-message: 'Stale pull request message'
        stale-issue-label: 'no-issue-activity'
        stale-pr-label: 'no-pr-activity'
```

Proposed:
```yaml
name: Mark stale issues and pull requests
permissions:
  contents: read
on:
  schedule:
    - cron: '43 7 * * *'
  workflow_dispatch:
jobs:
  stale:
    runs-on: ubuntu-latest
    permissions:
      issues: write
      pull-requests: write
    steps:
      - uses: actions/stale@v10
        with:
          repo-token: ${{ secrets.GITHUB_TOKEN }}
          days-before-issue-stale: 60
          days-before-issue-close: 14
          days-before-pr-stale: 30
          days-before-pr-close: 14
          stale-issue-message: >-
            This issue has had no activity for 60 days. It will close after 14 more
            days unless updated or given the security or codex active-work label.
          close-issue-message: >-
            Closing after 74 days without activity. Reopen with current evidence
            if the work is still needed.
          stale-pr-message: >-
            This pull request has had no activity for 30 days. It will close after
            14 more days unless updated or given an exempt active-work label.
          close-pr-message: >-
            Closing after 44 days without activity. Open a current pull request
            when the branch is ready for review.
          stale-issue-label: 'no-issue-activity'
          stale-pr-label: 'no-pr-activity'
          exempt-issue-labels: 'security,codex'
          exempt-pr-labels: 'security,codex'
          exempt-all-issue-assignees: true
          exempt-all-pr-assignees: true
          remove-stale-when-updated: true
          operations-per-run: 100
```

Verification:
- Focused contract, YAML parse, and permission diff.

### Task 5 - Document validate publish and close

Sequence / dependencies:
- After Tasks 2-4.

Implementation notes:
- Required skill: write-jane-street-style-code before the docs edit and final Review Mode.
- Before-Edit Brief:
  - Behavior: Contributors can discover the automation and diagnostics contract.
  - Invariants: Do not claim external success before GitHub runs checks.
  - Boundary/API: Root README contributor guide.
  - Effects and failures: Docs only; external behavior is verified on the PR.
  - Tests and evidence: Focused contracts, YAML, JS, Java, XML, and PR checks.

#### Code Edit 5.1
- File: README.md
- Lines: after 183
- Action: add

Proposed:
```markdown
Repository Automation
---------------------

Pull requests and main run the Java 25 build on Ubuntu, macOS, and Windows.
CI caches Gradle state and retains Java, JavaScript, and Gradle diagnostic
reports for 14 days when a matrix job fails. Browser tests write JUnit XML
under website/build/test-results/jsTest/.

CodeQL analyzes Java changes and Dependency Review rejects newly introduced
high-or-critical vulnerable dependencies. Dependabot groups weekly Gradle and
GitHub Actions updates. Security and codex active work is exempt from the
documented stale windows.
```

Verification:
- Render README and run the validation commands below.

Execution checklist:
- Parse workflow and Dependabot YAML with PyYAML.
- Run node --test website/src/test/js/github-automation.test.js.
- Run .\gradlew.bat :website:jsTest --rerun-tasks and parse JUnit XML.
- Run .\gradlew.bat :website:test with isolated GRADLE_USER_HOME.
- Run git diff --check and semantic review.
- Push, open a PR closing #1144-#1150, wait for all checks, fix in-scope failures, squash-merge, and confirm closure.

## Code Changes

- github-automation.test.js: automation contracts (1.1).
- ci.yml and website/build.gradle.kts: caching and diagnostics (2.1-2.2).
- codeql.yml and dependency-review.yml: security checks (3.1-3.2).
- dependabot.yml and stale.yml: update/triage policy (4.1-4.2).
- README.md: contributor contract (5.1).

## Files and Modules

.github/workflows, .github/dependabot.yml, website/build.gradle.kts, website/src/test/js, README.md.

## Unit Testing

RED/GREEN focused Node contract, full jsTest with JUnit XML, and website Java tests.

## Local Testing

No Spring runtime behavior changes, so alternate-port bootRun is not applicable. Parse YAML, run focused/full suites, and verify external behavior through PR checks. Do not manufacture a failing main run only to prove artifacts.

## Validation

Matrix preserved; PR cache read-only; reports retained 14 days; security checks run; configuration parses; stale job only has issue/PR writes.

## Rollback or Recovery

Revert the squash merge or one workflow. If a GitHub feature is unavailable, keep that issue open and record the exact error instead of weakening policy. If Node JUnit fails, revert only 2.2 and reproduce on Node 24.

## Risks

External feature availability; formatting-sensitive regex tests; Node-owned JUnit serialization; report absence during early setup failure. Mitigations are PR evidence, semantic assertions, XML validity only, and if-no-files-found warning.

## Completion Criteria

- RED/GREEN evidence for each semantic change.
- YAML, focused Node, jsTest, website:test, XML, and diff checks pass.
- PR CI, CodeQL, and Dependency Review pass or affected issues remain open with an external blocker.
- PR merges, #1144-#1150 close, and Builder artifacts are updated and pushed.
