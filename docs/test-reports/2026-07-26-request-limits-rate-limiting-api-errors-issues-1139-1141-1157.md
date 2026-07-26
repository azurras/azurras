# Request Limits, Rate Limiting, and API Errors Issues 1139-1141 and 1157 Test Report

## Document Status

complete

## Story/Issue

- [christopherbell.dev #1139](https://github.com/azurras/christopherbell.dev/issues/1139) - configurable request-size limits and standard 413 responses
- [christopherbell.dev #1140](https://github.com/azurras/christopherbell.dev/issues/1140) - expiring, bounded rate-limit bucket state
- [christopherbell.dev #1141](https://github.com/azurras/christopherbell.dev/issues/1141) - standard retry and rate-limit response headers
- [christopherbell.dev #1157](https://github.com/azurras/christopherbell.dev/issues/1157) - domain-specific service exceptions and consistent API responses

## Branch

- Repository: `A:\Projects\christopherbell.dev-worktrees\request-limits-api-errors-1139-1141-1157`
- Branch: `codex/request-limits-api-errors-1139-1141-1157`
- Commit under test: `d70e05d9` (`Improve request limits and API errors`)
- Base: `a5dc7a6381dd507e96cc2e045930acacf88089d7`

## App / Environment

- App: `christopherbell.dev` packaged Spring Boot application
- Java: repository Java 25 toolchain
- Profile: `local`
- Alternate port: `8090`
- Base URL: `http://127.0.0.1:8090`
- Production port: `8080`; listener PID `20156` remained running and unchanged throughout local verification
- MongoDB: `mongodb://localhost:27017`
- Disposable database: `christopherbell_request_limits_test_20260726001412`
- Request-size override: `app.request-size.default-max=1KB`
- Runtime rate rule: one `POST /api/accounts/2024-12-15/login` token per `5s`, with `rate-limit.max-buckets=20`
- Isolated Gradle home: `A:\Projects\.gradle-codex-request-limits-api-errors`

## Local Run Details

The final packaged app was built with:

```powershell
$env:GRADLE_USER_HOME='A:\Projects\.gradle-codex-request-limits-api-errors'
.\gradlew.bat :website:bootJar --no-daemon --console=plain
```

The alternate-port process was started hidden with the equivalent command:

```powershell
java -jar website\build\libs\website.jar `
  --spring.profiles.active=local `
  --server.port=8090 `
  --spring.mongodb.database=christopherbell_request_limits_test_20260726001412 `
  --spring.mongodb.uri=mongodb://localhost:27017 `
  --app.scheduling.enabled=false `
  --app.request-size.default-max=1KB `
  --rate-limit.max-buckets=20 `
  --rate-limit.rules[0].name=auth-test `
  --rate-limit.rules[0].capacity=1 `
  --rate-limit.rules[0].window=5s `
  --rate-limit.rules[0].methods[0]=POST `
  --rate-limit.rules[0].paths[0]=/api/accounts/2024-12-15/login
```

The first diagnostic run used PID `55164`; it was stopped after finding the chunked early-reader defect. The corrected packaged app used PID `56392`. PID `56392` was stopped after the final checks. Runtime logs are build-owned files:

- `website/build/request-limits-runtime-20260726001412.log`
- `website/build/request-limits-runtime-20260726001412.err.log`
- `website/build/request-limits-runtime-20260726001812.log`
- `website/build/request-limits-runtime-20260726001812.err.log`

The exact disposable database was dropped with `mongosh` after the corrected run. The result was `{ ok: 1, dropped: 'christopherbell_request_limits_test_20260726001412' }`. Final listener enumeration showed only production port `8080`, PID `20156`; no listener remained on `8090`.

## Test Cases

### 1. Known-length oversized JSON request

Sent a 2,048-byte JSON-content request against the login endpoint while the configured ordinary limit was 1KB. Verified rejection occurs before rate limiting, authentication, validation, or controller work.

### 2. Unknown-length chunked oversized JSON request

Opened a raw TCP connection and sent HTTP/1.1 chunked transfer encoding with one hexadecimal `800` chunk containing 2,048 bytes. Verified the filter pre-reads only the configured limit plus one byte, rejects the body before a downstream JSON parser can stop early, and returns the standard envelope.

### 3. Exhausted five-second login rate rule

After a fresh process start, sent the same small login request twice. Verified the first request consumed the only token and reached controller validation, while the second was rejected at the rate-limit filter with actionable headers and the standard API envelope.

### 4. Automated RED/GREEN and repository regression

Witnessed compile RED for the missing typed request/error-writer APIs, ten exact service RED failures while generic wrappers remained, and a focused RED for an unknown-length downstream reader that consumed only one byte. After implementation, ran focused tests and the full clean repository gate.

## Data Sent

### Known-length 413

- Method/URL: `POST http://127.0.0.1:8090/api/accounts/2024-12-15/login`
- Header: `Content-Type: application/json`
- Framing: known `Content-Length` from `ByteArrayContent`
- Body: 2,048 zero bytes; body content was not reflected in the response

### Chunked 413

- Request line: `POST /api/accounts/2024-12-15/login HTTP/1.1`
- Headers: `Host: 127.0.0.1:8090`, `Content-Type: application/json`, `Transfer-Encoding: chunked`, `Connection: close`
- Chunk: `800\r\n` followed by 2,048 `x` bytes, then `0\r\n\r\n`

### Rate limiting

- Method/URL: two consecutive `POST http://127.0.0.1:8090/api/accounts/2024-12-15/login` requests
- Header: `Content-Type: application/json; charset=utf-8`
- Body: `{"email":"x","password":"x"}`

## Response Received

### Known-length 413

- Status: `413`
- Content-Type: `application/json; charset=UTF-8`
- Body:

```json
{"messages":[{"code":"REQUEST_TOO_LARGE","description":"The request body exceeds the allowed size."}],"payload":null,"requestId":null,"success":false}
```

### Chunked 413

- Status line: `HTTP/1.1 413`
- Body contained `REQUEST_TOO_LARGE`
- The corrected request did not consume the rate-limit token

### First small login request

- Status: `400`
- Body: standard `success=false` response with `REQUEST_ERROR` for the intentionally malformed email
- This proves the request passed the rate-limit filter and reached controller validation

### Second small login request

- Status: `429`
- `Retry-After: 5`
- `X-RateLimit-Limit: 1`
- `X-RateLimit-Remaining: 0`
- `X-RateLimit-Reset: 1785043132`
- Body:

```json
{"messages":[{"code":"RATE_LIMITED","description":"Too many requests. Try again later."}],"payload":null,"requestId":null,"success":false}
```

## Pass / Fail

| Test case | Result | Reason |
| --- | --- | --- |
| Known-length oversized JSON | PASS | Returned standard, redacted `413 REQUEST_TOO_LARGE` before downstream work. |
| Unknown-length chunked oversized JSON | PASS | Corrected run returned `HTTP/1.1 413` even though the malformed JSON parser could otherwise stop early. |
| Exhausted login rate rule | PASS | First request reached validation; second returned `429` with retry, limit, remaining, reset, and standard body. |
| Rate bucket expiry/bounds | PASS | Focused tests cover sliding expiry, active-cardinality hard bound, access refresh, and extreme-duration overflow. |
| Typed service failures | PASS | Ten targeted operational paths preserve causes in `ServiceUnavailableException` or `InternalServiceException`; safe global mappings pass. |
| Full repository regression | PASS | `cleanTest check` completed successfully on the final tree. |
| Cleanup and production isolation | PASS | PID `56392` stopped, disposable database dropped, port `8090` released, and production PID `20156` remained on `8080`. |

## Evidence

- Initial focused compile RED: missing `ApiErrorResponseWriter` and typed constructor contracts.
- Service RED: `106 tests completed, 10 failed`; the ten failures were exactly the generic service wrapper paths.
- Unknown-length early-reader RED: `1 test completed, 1 failed` for `unknownLengthOverflowIsRejectedWhenDownstreamReadsOnlyOneByte`.
- Final focused matrix: all request-size, rate-limit, configuration, exception-handler, account, vehicle, and restaurant tests passed.
- Final full command: `.\gradlew.bat cleanTest check --no-daemon --console=plain`.
- Final full result: `BUILD SUCCESSFUL in 1m 38s`; 1,074 website tests and 95 library tests, 1,169 total, zero failures, three expected skips; `website:verifySensorRuntime` and `website:check` passed.
- Runtime root smoke: `GET http://127.0.0.1:8090/` returned `200` before acceptance requests.
- Corrected runtime evidence: known-length `413`, raw chunked `413`, first small login `400`, second small login `429` with all four expected rate headers.
- Cleanup evidence: MongoDB returned `{ ok: 1, dropped: 'christopherbell_request_limits_test_20260726001412' }`; final listener enumeration retained only `8080 -> PID 20156`.

## Bugs / Follow-ups

- Resolved during testing: the first raw chunked request returned `400 REQUEST_ERROR` because a malformed JSON parser stopped before the streaming wrapper observed limit-plus-one bytes. A focused RED test reproduced that downstream early-reader path. Unknown-length bodies now pre-read at most limit-plus-one bytes and replay only accepted bodies; the corrected raw request returned `413`.
- Resolved during full regression: MVC slices initially failed because the servlet writer used the legacy Jackson 2 mapper type. Switching to the application-native Jackson 3 `tools.jackson.databind.ObjectMapper` restored all 13 affected tests.
- No known acceptance gap remains. Production deployment is intentionally deferred until the PR is merged and CI is green.
