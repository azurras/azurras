# Public Content Issues 1131-1137 Spoke Review

## Document Status

complete

## Reviewed Spoke

- Repository: `https://github.com/azurras/christopherbell.dev.git`
- Worktree: `A:\Projects\christopherbell.dev-worktrees\public-content-1131-1137`
- Branch: `codex/public-content-1131-1137`
- Base: `b6c361d1d916337679a37f04caa46c3475215e71`
- Reviewed head: `4108c5c6f5adf5877f247c2cff4cf543fd7eb1cd`
- Pull request: [azurras/christopherbell.dev#1251](https://github.com/azurras/christopherbell.dev/pull/1251)
- Issues: `#1131`, `#1132`, `#1133`, `#1134`, `#1135`, `#1136`, and `#1137`

## Scope Reviewed

- Anonymous GET-only blog/photo API boundaries and standard response envelopes.
- Browser normalizers, safe text rendering, unsupported tag removal, and gallery alt-text fallbacks.
- Photography usage route/link.
- The Bell links, favicon, image assets, insecure sources, and stray markup.
- Pinned self-hosted Bootstrap 5.3.3, CSP narrowing, template deduplication, and Dependency Review coverage.
- Tests, package documentation, local HTTP/browser evidence, final committed diff, and worktree hygiene.

## Findings

No remaining Blocker or Warning findings.

The initial review found one blocker: `PhotoProperties` bound `photo-properties.photos` while `application.yml` supplied `photo-properties.images`, leaving the live API empty despite 12 configured entries. Commit `4108c5c6f5adf5877f247c2cff4cf543fd7eb1cd` aligns the key, adds a configuration-context regression with witnessed RED/GREEN evidence, and treats configured `n/a` descriptions as missing alt text so the photo name is used. Independent re-review found the blocker closed and no new findings.

## Validation Checked

- Focused Java public-content suite: 29 passed before review.
- Configuration-binding RED: `PhotoPropertiesConfigurationTest` failed on a null collection before the YAML correction.
- Configuration-binding GREEN: `PhotoPropertiesConfigurationTest` and `PhotoControllerTest` passed after correction.
- Focused Node public-content suite: 4 passed after witnessed RED failures for both payload normalization and the `n/a` alt sentinel.
- Authoritative final `cleanTest + check --max-workers=1 --no-watch-fs`: `BUILD SUCCESSFUL`; 1003 Java tests, 0 failures, 3 skipped; 199 JavaScript tests; `bootJar` and `verifySensorRuntime` passed.
- Live alternate-port API: `GET /api/photo/v1` returned `success=true` with 12 configured photos; the first JPEG returned `200 image/jpeg`, length `4770189`.
- Browser: 12 gallery images rendered; name fallbacks replaced 11 `n/a` descriptions, the one real description remained, the usage link was present, and console logs were empty.
- Static checks: JavaScript syntax, `git diff --check`, recursive no-Bootstrap-CDN scan, archive local-asset scan, and no-insecure-image scan passed.
- Dependency insight selected `org.webjars:bootstrap:5.3.3`; Dependency Review passed on both PR revisions.
- Port `8090` was stopped; production PID `26680` on `8080` returned `200` after final local testing.

## House-Style Review

The implementation follows the repository-native Java, JavaScript, template, configuration, and test conventions while preserving explicit public/private boundaries. The browser renderers own copied collections, use text DOM APIs, isolate response-shape normalization, and partition fallback behavior. The configuration-binding test covers the boundary that controller mocks cannot prove.

## Risks

- Bootstrap advances legacy pages from 5.0.2/5.3.3 CDN mixtures to one local 5.3.3 artifact; focused browser checks cover the public pages in this batch, and the full cross-platform suite covers shared regressions.
- The local profile can run unrelated startup jobs unless explicitly disabled. Final gallery verification used `WFL_RESTAURANT_IMPORT_MONTHLY_ENABLED=false`.
- `gradlew.bat` shows a checkout-only LF-to-CRLF difference in this worktree. It is unstaged and absent from both spoke commits.

## Requested Changes

None remaining.

## Merge Readiness

Ready to merge after the rerun GitHub CI and CodeQL checks on head `4108c5c6` pass.
