# 2026-07-25 - Plan GitHub Automation Issues 1144-1150

## Request

Continue the approved campaign to complete every open `azurras/christopherbell.dev` issue without routine approval pauses, beginning with the GitHub automation portion of Batch 1.

## Work Completed

- Created `docs/implementation-plans/2026-07-25-github-automation-issues-1144-1150.md` for issues #1144-#1150.
- Defined artifact-native RED/GREEN coverage for Gradle caching, failed-run diagnostics, CodeQL, Dependency Review, Dependabot grouping, and stale-policy permissions and windows.
- Split the broader deployment/configuration batch so this independent automation work can be reviewed, rolled back, merged, and closed as one coherent pull request.
- Updated the central work ledger with the ready plan.

## Decisions

- Execute inline without subagents, consistent with the user's autonomous campaign authorization.
- Keep each GitHub automation concern in its native configuration file and add a Node built-in filesystem contract test rather than introducing an npm dependency.
- Preserve console test output while adding JUnit XML for artifact collection.
- Use current supported major action tags verified from official action documentation.
- Parse YAML into structured Jackson trees in JUnit rather than relying on formatting-sensitive Node regular expressions; include the checkout step shown by the official Dependency Review installation contract.

## Validation

- `validate_implementation_plan.py` passed.
- Placeholder scan and `git diff --check` passed.
- Execution-readiness review found no blockers. Residual risk is limited to GitHub-hosted feature availability and action behavior, which the pull request checks must prove.
- After remote main advanced, the baseline was refreshed at `ea2ba7ea4c4ab1b71f172a29dd994e8375507675`: `:website:test` and all 176 `jsTest` cases passed. The reviewed plan was updated before implementation to use parsed YAML assertions and the current README insertion line.
- Full trusted issue-body intake corrected the stale contract before production edits: workflow-level permissions are empty, the job receives only issue and pull-request writes, assigned and milestone issues are directly exempt, and `pinned`/`roadmap` protection labels cover action/stale's lack of native pin-metadata support.
- Final diff review made `jackson-dataformat-yaml` an explicit test dependency so the parsed-YAML contract does not rely on Springdoc's transitive classpath.
- PR #1241 proved that GitHub default setup rejects advanced-workflow SARIF uploads. The corrective design preserves all three default-setup languages in the checked-in matrix, builds Java manually, and only then switches default setup off through GitHub's supported API.

## Completion

- Implemented and pushed commits `0b7c117f` and `86e7442d` on `codex/github-automation-1144-1150`.
- Merged PR [#1241](https://github.com/azurras/christopherbell.dev/pull/1241) as squash commit `88144134290e5f690c048cb4945db531b8ef17c9`.
- GitHub closed #1144, #1145, #1146, #1147, #1148, #1149, and #1150 at merge time.
- Created and verified repository labels `pinned` and `roadmap` for stale-policy protection.
- Switched CodeQL from default setup to the checked-in advanced workflow through the supported repository API after preserving Actions, Java/Kotlin, and JavaScript/TypeScript coverage.

## Final Validation

- RED: all five parsed-YAML configuration contracts failed before implementation; the expanded CodeQL matrix contract also failed before the hosted fix.
- GREEN: the focused five-test JUnit contract passed.
- `gradlew build --rerun-tasks --no-daemon` passed on the rebased final tree; the browser JUnit report parsed with a `testsuites` root and all 176 browser tests passed.
- Hosted checks passed on Ubuntu, macOS, Windows, Dependency Review, and all CodeQL language jobs.
- Independent review found no Critical, Important, or Minor findings.
- Local Spring app testing and a Builder test report were not applicable because this batch changed repository automation only and did not change runtime application behavior.

## Follow-up

- Continue the autonomous campaign with the remaining 51 open issues, beginning with the next production/deployment/configuration sub-plan.

## Current State and Follow-up

- Plan status is `ready-for-execution`.
- Next create `codex/github-automation-1144-1150` in a fresh sibling worktree, invoke the required implementation skills, witness RED, implement, run the full relevant suites, publish a pull request, wait for checks, merge, and close #1144-#1150.
