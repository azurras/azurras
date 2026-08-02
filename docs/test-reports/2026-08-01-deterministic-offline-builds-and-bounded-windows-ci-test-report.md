# Deterministic Offline Builds and Bounded Windows CI Test Report

## Document Status

complete

## Story/Issue

- `azurras/christopherbell.dev#1302` - deterministic artifact versions
- `azurras/christopherbell.dev#1303` - cached/offline sensor resources
- `azurras/christopherbell.dev#1304` - Windows Pester suites in CI
- `azurras/christopherbell.dev#1305` - CI cancellation and timeouts
- PRs #1330-#1335

Only issue text from trusted author `azurras` controlled scope; the issues had no comments or attachments.

## Branch

- Final branch: `codex/issues-1302-1305-exact-gradle-home-fix-20260801`
- Final branch head: `b727903ddb91cfebe32285ae966edc866b01e802`
- Final merge: `ad8744f79b42597c7ae53f7f83e9190eb295e491`
- Deployed direct descendant: `c4d60ce0c92281c201d063cfd6a07563f4a7b230`
- Isolated worktree: `A:\Projects\christopherbell.dev-worktrees\issues-1302-1305-ci-date-fix-20260801`
- The dirty authoritative checkout remained unchanged.

## App / Environment

- Spring Boot website on the native Windows production host.
- Isolated Gradle home: `A:\Projects\christopherbell.dev-gradle\issues-1302-1305-20260729`.
- Pester 5.9.0 under PowerShell 7 and Windows PowerShell 5.1.
- Protected deployment Gradle home: `C:\ProgramData\christopherbell.dev\gradle-home`; its ACLs were not weakened.
- Local runtime: `http://127.0.0.1:8080`.
- Public runtime: `https://christopherbell.dev/` and `https://www.christopherbell.dev/`.
- The protected pipeline validates a candidate before rotating the port-8080 listener.

## Local Run Details

Focused RED/GREEN checks covered deterministic versioning, sensor cache resolution, CI workflow contracts, date-stable expiration fixtures, fixed production ACL principals, and the protected build-context truth table. Ordinary Windows builds retained all Pester suites. A clean production-shaped build omitted only Pester while retaining Java, JavaScript, version, sensor, packaging, and other checks.

The final ordinary `gradlew.bat --no-daemon build --console=plain` succeeded in 2m47s. The protected deployment then built and validated current main before rotating production to `c4d60ce0c92281c201d063cfd6a07563f4a7b230`.

## Test Cases

1. Resolve versions repeatedly for one commit and reject invalid release metadata.
2. Exercise cold-online, cached-offline, corrupt-checksum, unavailable-writer, and no-partial sensor behavior.
3. Run three Pester suites through `build` and parse NUnit XML.
4. Verify PR cancellation, preserved main runs, job timeouts, pinned Pester, reports, and non-Windows exclusion.
5. Reproduce/fix date-bound post tests and the ACL assertion under `SYSTEM`.
6. Qualify only the exact protected Windows Gradle home; reject invalid markers and lookalike paths.
7. Run full regression, PR/main CI, CodeQL, protected deployment, endpoints, assets, access control, listener, and services.

## Data Sent

- Verification inputs included repeated commit identity, valid/invalid release overrides, empty/warm/corrupt sensor caches, offline mode, and an unavailable writer.
- Context inputs included the exact protected home, absent identity strings, invalid markers, non-Windows contexts, ordinary homes, and `A:\scratch\christopherbell.dev\gradle-home`.
- `GET http://127.0.0.1:8080/`, `/blog`, `/wfl`, `/robots.txt`, `/sitemap.xml`, `/favicon.ico`, `/.well-known/nodeinfo`, `/nodeinfo/2.1`, `/actuator/health/liveness`, `/actuator/health/readiness`, and `/api/v1/command-center/snapshot`.
- `GET https://christopherbell.dev/`, `GET https://www.christopherbell.dev/`, and one SHA-versioned CSS asset.

## Response Received

- Full Windows build: success; 1,660 Java tests with 0 failures/errors and 3 skipped; 289 JavaScript tests with 0 failures; 150 Pester tests with 0 failures.
- Pester totals: operations Windows PowerShell 5.1: 38; operations PowerShell 7: 38; worker PowerShell 7: 74.
- Local public routes, liveness, and readiness status code: 200. Anonymous command-center status code: 403.
- Favicon status code: 200 with 32,038 bytes.
- Apex and `www` root status code: 200; each response body contained deployed SHA `c4d60ce0c92281c201d063cfd6a07563f4a7b230`.
- Versioned CSS status code: 200; cache header was `public, max-age=31536000, immutable`.
- Port 8080 listener: Java PID 39812. `ChristopherBellDev`, `ChristopherBellMediaWorker`, `Cloudflared`, and `MongoDB`: Running/Automatic.
- Main CI runs 30726222833 and 30726230123: success. CodeQL 30726230146: success.

## Pass / Fail

- #1302 deterministic artifact identity: PASS.
- #1303 bounded checksum-verified cached/offline packaging: PASS.
- #1304 pinned Windows Pester/NUnit CI: PASS.
- #1305 concurrency and timeouts: PASS.
- Full regression, CI/CodeQL, protected deployment, and production acceptance: PASS.

## Evidence

- PRs: https://github.com/azurras/christopherbell.dev/pull/1330, https://github.com/azurras/christopherbell.dev/pull/1331, https://github.com/azurras/christopherbell.dev/pull/1332, https://github.com/azurras/christopherbell.dev/pull/1333, https://github.com/azurras/christopherbell.dev/pull/1334, https://github.com/azurras/christopherbell.dev/pull/1335
- Final implementation CI: https://github.com/azurras/christopherbell.dev/actions/runs/30726222833
- Deployed descendant CI: https://github.com/azurras/christopherbell.dev/actions/runs/30726230123
- Deployed descendant CodeQL: https://github.com/azurras/christopherbell.dev/actions/runs/30726230146
- Spec: `docs/specs/2026-07-29-deterministic-offline-builds-and-bounded-windows-ci.md`
- Plan: `docs/implementation-plans/2026-07-29-deterministic-offline-builds-and-bounded-windows-ci-implementation-plan.md`
- Production acceptance: 2026-08-01 America/Chicago.

## Bugs / Follow-ups

Merged-main/production evidence exposed user-scoped Pester availability, July fixture expiration, a contradictory SYSTEM assertion, and unstable scheduled-task username sources. Each received a focused RED/GREEN fix-forward PR. The final exact ACL-protected path boundary passed all checks and production advanced. No unresolved product gap remains.
