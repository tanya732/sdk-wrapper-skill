# Stage 6: Test Generation — Prompt Template

## Trigger

User has approved Stage 5 generated code (ideally after confirming it compiles).

## System Prompt

You are executing **Stage 6: Test Generation** of the SDK Wrapper Skill.

Generate a comprehensive test suite for the framework SDK. Tests must cover: unit tests, integration tests, and edge cases. The test suite should give maintainers confidence to merge PRs and release versions.

## Test Strategy

### Layer 1: Unit Tests (Mock Core SDK)

For every public method in the wrapper:

```
Test: {MethodName}_success
  Given: Core SDK returns successful response
  When:  Wrapper method is called with valid inputs
  Then:  Response is correctly adapted to framework types

Test: {MethodName}_error_{errorType}
  Given: Core SDK throws {ErrorType}
  When:  Wrapper method is called
  Then:  Error is correctly mapped to framework error

Test: {MethodName}_nullInput
  Given: Null/empty/missing required input
  When:  Wrapper method is called
  Then:  Appropriate validation error thrown immediately
```

**Mocking approach:**

- Mock the core SDK client (not HTTP calls)
- This ensures wrapper logic is tested in isolation
- Use framework-standard mocking library

### Layer 2: Configuration Tests

```
Test: config_allRequired
  Given: All required config keys present
  When:  Module initializes
  Then:  Core SDK client created with correct values

Test: config_missingRequired
  Given: Required config key missing
  When:  Module initializes
  Then:  Clear error message naming the missing key

Test: config_defaults
  Given: Only required keys, no optional keys
  When:  Module initializes
  Then:  Optional settings use documented defaults

Test: config_envVarOverride
  Given: Config file value AND env var both set
  When:  Module initializes
  Then:  Env var takes precedence

Test: config_invalidValue
  Given: Config key with invalid value (e.g., negative timeout)
  When:  Module initializes
  Then:  Validation error with helpful message
```

### Layer 3: Integration Tests (Framework Test Server)

Using the framework's test utilities:

```
Test: fullFlow_{useCase}
  Given: Framework test server running with SDK configured
  When:  HTTP request hits endpoint that uses SDK
  Then:  Core SDK is called correctly and response is proper

Test: middleware_authentication
  Given: Auth middleware is registered
  When:  Request without token
  Then:  401 response with correct error body

Test: middleware_rateLimiting
  Given: Core SDK returns 429
  When:  Wrapper handles the error
  Then:  Framework returns 429 with Retry-After header
```

### Layer 4: Edge Cases

```
Test: concurrency_threadSafety
  Given: Multiple threads/requests calling wrapper simultaneously
  When:  All complete
  Then:  No data corruption, no shared state issues

Test: timeout_coreSDK
  Given: Core SDK call exceeds configured timeout
  When:  Timeout fires
  Then:  Appropriate timeout error returned

Test: serialization_specialChars
  Given: Response contains unicode, special characters
  When:  Deserialized through wrapper
  Then:  Characters preserved correctly

Test: pagination_iteration
  Given: Core SDK returns paginated results
  When:  Wrapper iterates
  Then:  All pages retrieved, order preserved

Test: lifecycle_reinitialize
  Given: SDK initialized, used, then application restarted
  When:  New instance created
  Then:  Works correctly (no stale state)
```

## Test Utilities to Generate

### Mock Factories

```
{SdkName}Mocks.{ext}
  - mockAuthClient(responses...)
  - mockManagementClient(responses...)
  - mockErrorResponse(statusCode, errorBody)
  - mockSuccessResponse(body)
```

### Test Fixtures

```
{SdkName}Fixtures.{ext}
  - validConfig()
  - minimalConfig()
  - invalidConfig()
  - sampleUser()
  - sampleToken()
  - sampleErrorResponse()
```

### Test Helpers

```
{SdkName}TestHelper.{ext}
  - withTestApp(config, test) → runs test with framework test server
  - assertErrorResponse(response, expectedStatus, expectedMessage)
  - assertSuccessResponse(response, expectedBody)
```

## Coverage Targets

| Category        | Target | Rationale                                 |
| --------------- | ------ | ----------------------------------------- |
| Line Coverage   | ≥ 85%  | High confidence for releases              |
| Branch Coverage | ≥ 75%  | Error paths exercised                     |
| Public API      | 100%   | Every public method has at least one test |
| Error Paths     | 100%   | Every error mapping tested                |
| Configuration   | 100%   | Every config key tested (valid & invalid) |

## Output Format

Generate all test files. After generation, provide a summary:

```markdown
## Test Suite Summary

| Category            | Tests | Files |
| ------------------- | ----- | ----- |
| Unit Tests          | N     | N     |
| Configuration Tests | N     | N     |
| Integration Tests   | N     | N     |
| Edge Case Tests     | N     | N     |
| **Total**           | **N** | **N** |

### Test Utilities Generated

- MockFactory: {file}
- Fixtures: {file}
- TestHelper: {file}

### Run Tests

{command to run tests}
```

## Completion Signal

> **Stage 6 Complete.** I've generated the test suite.
>
> **Summary:** [N tests across N files]
>
> Please:
>
> 1. Run the tests: `{test command}`
> 2. Check if any tests fail (expected: all pass with mocked core SDK)
> 3. Let me know if any test scenarios are missing
>
> Once approved, we'll proceed to **Stage 7: Update & Migration Strategy**.
