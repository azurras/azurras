# Issue 1261 True-Error Logging Hardening

## Document Status

Ready for review.

## Purpose

Resolve `azurras/christopherbell.dev` issue #1261 and the related production log flood by making log severity reflect operational meaning: expected client failures must not appear as stack-traced errors, while unexpected server and infrastructure failures must retain actionable diagnostics.

## Background

Issue #1261 reports that `ControllerExceptionHandler` exposes raw framework exception messages and writes routine HTTP 4xx outcomes as full `ERROR` stack traces. The approved campaign spec and account-security implementation plan already require stable public request errors, server-side diagnostics, and preservation of unexpected-500 causes.

Live inspection of the authenticated Mission Control log stream on production commit `8405cd77d0f1743fe33d70cc80b47e37048090a0` confirmed a second contributor. A bounded initial sample contained 14 `ERROR` headers and 1,763 stack-frame lines over 107 seconds. Every error belonged to the same Spring Security/Tomcat failure family:

- `AuthorizationDeniedException: Access Denied` during a servlet async dispatch.
- A follow-on failure because the streaming response was already committed.
- A container error dispatch to `/error`, which caused another large trace.

The pair continued approximately every 16 seconds. The deployed `SecurityConfig` authorizes every dispatcher type through the ordinary request rules and does not define an explicit `ASYNC` or `ERROR` dispatcher policy. Spring Security documents that authorization runs on every dispatch and that dispatcher-type mismatches can produce unexpected access denial. The repository's `StreamingResponseBody` endpoints make the async path reachable.

The authoritative checkout at `A:\Projects\christopherbell.dev` contains extensive unrelated changes and must remain untouched. Implementation belongs in the existing isolated worktree `A:\Projects\christopherbell.dev-worktrees\all-open-issues-20260729` on `codex/all-open-issues-20260729`.

## Goals

- Stop the repeated async/error-dispatch access-denied cascade at its source.
- Ensure expected caller-correctable HTTP 4xx outcomes never produce `ERROR` stack traces.
- Return stable, non-sensitive client descriptions for parsing, binding, validation, media-type, response-type, authentication, authorization, conflict, and not-found failures.
- Preserve full causal diagnostics for unexpected HTTP 5xx and genuine infrastructure or operational failures.
- Add automated and runtime evidence that distinguishes expected client failures from true server errors.
- Complete the existing issue #1261 delivery loop through local runtime testing, PR/CI, merge, production verification, issue closure, and Builder closeout.

## Non-Goals

- Suppress broad Spring, Tomcat, or application logger categories.
- Hide genuine server, infrastructure, scheduled-job, persistence, or integration failures.
- Treat Mission Control UI filtering as a substitute for correcting log producers.
- Redesign the API response envelope, authentication model, media endpoints, or command-center log reader.
- Reclassify unrelated warning or informational messages without direct evidence.
- Modify or clean the dirty authoritative checkout.

## Requirements

### Dispatcher authorization

- The initial `REQUEST` dispatch must continue through the complete existing authentication and authorization policy.
- Subsequent `ASYNC` and `ERROR` dispatches must not repeat request authorization after the original request has already passed the security boundary.
- The change must use explicit dispatcher-type matchers rather than disabling authorization filtering for all dispatcher types.
- Security headers and the remainder of the Spring Security filter chain must remain active.
- Protected streaming endpoints must remain inaccessible to unauthenticated or unauthorized initial requests.

### Client-safe API errors

- Framework parsing, binding, validation, unsupported-media-type, unacceptable-response-type, and response-status failures must use stable public descriptions selected by status or error category.
- Raw parser, converter, reflection, property, validation-internal, and framework exception messages must not cross the HTTP boundary.
- Existing response envelope and status contracts must remain compatible unless issue #1261 explicitly requires safer wording.

### Log severity contract

- Expected HTTP 4xx outcomes must not be logged at `ERROR` and must not include stack traces.
- Security-relevant client failures such as 401, 403, and 429 may produce one bounded structured warning containing safe category and status fields, but never credentials, tokens, raw request bodies, or a stack trace.
- Ordinary caller-correctable failures such as 400, 404, 406, 409, and 415 may produce one bounded `DEBUG` or `INFO` record when operationally useful; they may also be silent when an existing request/audit signal already provides sufficient evidence.
- Unexpected 5xx failures must remain `ERROR` and retain the exception cause server-side while returning a stable generic public description.
- Known infrastructure failures mapped to 503 must remain operationally visible with their cause and must not be downgraded merely because the public response is stable.

### Mission Control behavior

- Mission Control must continue to display the configured, bounded, redacted application log.
- No package-level suppression or display-only filtering may be used to claim that the producer-side error flood is fixed.
- The existing level selector may remain unchanged; after remediation, selecting `ERROR` must represent true server or operational failures rather than expected client traffic or async redispatch artifacts.

## Proposed Approach

1. Add explicit `DispatcherType.ASYNC` and `DispatcherType.ERROR` permit rules before URL matchers in `SecurityConfig`. This preserves authorization on the original request while preventing a committed streaming response from being re-authorized during completion or error rendering.
2. Add a security integration test that first proves an unauthenticated protected streaming request is denied, then proves an authorized async request can complete and redispatch without `AuthorizationDeniedException`, committed-response failure, or `/error` cascade.
3. Refactor `ControllerExceptionHandler` around a small explicit mapping from known framework exceptions/statuses to stable public descriptions and bounded log classifications.
4. Emit one-line, non-stack-traced records for expected 4xx cases only at the approved lower level; keep full exception logging for unexpected 5xx and known infrastructure failures.
5. Extend controller-advice contract tests for malformed JSON, validation, unsupported media type, unacceptable response type, access denied, conflict, not found, service unavailable, and unexpected exceptions.
6. Capture logging output in tests to prove expected 4xx paths emit no `ERROR` stack trace and unexpected 500 paths still emit one causal `ERROR`.

## Files and Modules Involved

- `website/src/main/java/dev/christopherbell/configuration/security/SecurityConfig.java`
- `website/src/test/java/dev/christopherbell/configuration/SecurityConfigTest.java`
- `website/src/test/java/dev/christopherbell/configuration/security/AsyncDispatcherSecurityIntegrationTest.java` (new focused integration test).
- `cbell-lib/src/main/java/dev/christopherbell/libs/api/controller/ControllerExceptionHandler.java`
- `cbell-lib/src/test/java/dev/christopherbell/libs/api/controller/ControllerExceptionHandlerTest.java`
- Existing controller slice tests only where public error descriptions or logging expectations change.

## Validation Plan

### Automated validation

- Record a failing test for the async redispatch cascade before changing security configuration.
- Record failing controller-advice and captured-log tests before changing exception mapping.
- Run focused `cbell-lib` and `website` tests during development.
- Run `:website:check` with an isolated `GRADLE_USER_HOME` before publication.
- Run repository format, lint, and policy checks required by the current mainline.

### Local runtime validation

- Start the production-profile application on a non-8080 port using the established safe local verification workflow.
- Exercise an authenticated streaming response and confirm completion produces no access-denied or committed-response cascade.
- Send representative malformed JSON, validation, unsupported-media, access-denied, not-found, and unexpected-failure requests with non-sensitive fixtures.
- Verify 4xx response bodies contain only stable descriptions and produce no `ERROR` stack traces.
- Verify a controlled unexpected 500 returns the generic public response and produces one causal `ERROR` server-side.
- Verify `/` and the authenticated Mission Control snapshot/log endpoints still work.

### Publication and production validation

- Commit implementation changes on the isolated spoke branch and open a focused pull request that closes #1261.
- Require repository CI and CodeQL to pass before merge.
- Merge only after required checks pass.
- Deploy through the existing native Windows production workflow.
- Confirm the live SHA, service state, local and public `/` responses, authenticated Mission Control health, and absence of the prior repeating async-dispatch signature.
- Keep issue #1261 open until merged production behavior is verified and closure evidence is recorded.

## Acceptance Criteria

- An unauthorized initial request to a protected streaming endpoint is still denied.
- An authorized streaming response completes without a second-dispatch `AuthorizationDeniedException` or committed-response error cascade.
- Expected 4xx cases do not produce `ERROR` stack traces.
- Stable public error descriptions contain no raw framework/parser details.
- Unexpected 500 and genuine infrastructure failures retain causal `ERROR` diagnostics.
- Focused tests, full checks, alternate-port runtime verification, required CI, merge, and production verification all pass.
- GitHub issue #1261 is closed with commit, test-report, PR, CI, merge, deployment, and runtime evidence.

## Risks and Mitigations

- Permitting the wrong dispatcher scope could weaken authorization. Limit permit rules to `ASYNC` and `ERROR`, keep original `REQUEST` authorization unchanged, and prove initial protected requests remain denied.
- Downgrading failures too broadly could conceal defects. Use explicit known 4xx mappings and retain the fallback 5xx path as causal `ERROR` logging.
- Captured-log tests can become brittle. Assert severity, exception presence, and safe category rather than full rendered lines.
- Production traffic could differ from local reproduction. Verify the exact live error signature disappears after deployment while service and endpoint health remain intact.

## Open Questions

None. The source-level remediation policy and production verification boundary were approved on 2026-07-29.
