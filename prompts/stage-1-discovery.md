# Stage 1: Core SDK Discovery — Prompt Template

## Trigger

User provides a core SDK repository URL or package identifier.

## System Prompt

You are executing **Stage 1: Core SDK Discovery** of the SDK Wrapper Skill.

Given the core SDK repository at `{{REPO_URL}}`, perform an exhaustive analysis of its public API surface, internal architecture, and conventions.

## Analysis Checklist

### 1. Project Structure

- [ ] Identify build system (Maven, Gradle, npm, pip, etc.)
- [ ] Detect monorepo vs single-repo
- [ ] Map directory structure and module organization
- [ ] Identify code generation markers (e.g., Fern, OpenAPI, Swagger Codegen)
- [ ] Note any handwritten vs auto-generated code boundaries

### 2. Public API Surface

For each public class/module/function:

- [ ] Fully qualified name
- [ ] Constructor / factory method signatures
- [ ] Public method signatures with parameter types and return types
- [ ] Static vs instance methods
- [ ] Deprecated methods (note replacement)
- [ ] Overloaded method variants

### 3. Design Patterns

- [ ] **Initialization:** Builder, Factory, Constructor, Static Factory
- [ ] **Configuration:** Builder stages, Config objects, Environment variables
- [ ] **Client structure:** Monolithic, Hierarchical sub-clients, Fluent API
- [ ] **Immutability:** Immutable types, defensive copies, final fields
- [ ] **Lazy initialization:** Memoized suppliers, lazy singletons
- [ ] **Visitor pattern:** For enums, discriminated unions
- [ ] **Pagination:** Iterator pattern, cursor-based, offset-based

### 4. Error Handling

- [ ] Exception hierarchy tree (draw full inheritance)
- [ ] Checked vs unchecked exceptions
- [ ] Error response models (status codes, error bodies)
- [ ] Retry behavior (built-in retry, backoff strategies)
- [ ] Rate limiting handling (429 responses, Retry-After headers)

### 5. Async/Sync Model

- [ ] Primary paradigm: synchronous, async (CompletableFuture, Promise, Task), reactive (Flux, Observable)
- [ ] Blocking vs non-blocking I/O
- [ ] Thread-safety guarantees (thread-safe clients, connection pooling)
- [ ] Callback patterns

### 6. HTTP & Networking

- [ ] HTTP client library (OkHttp, Apache HttpClient, Fetch, Requests, HttpClient)
- [ ] Request/response interceptors
- [ ] Authentication mechanism (API keys, OAuth tokens, JWT)
- [ ] Base URL configuration
- [ ] Timeout configuration (connect, read, write)
- [ ] Proxy support
- [ ] Connection pooling

### 7. Serialization

- [ ] JSON library (Jackson, Gson, System.Text.Json, etc.)
- [ ] Custom serializers/deserializers
- [ ] Null handling strategy (Optional, @Nullable, null-safe)
- [ ] Date/time serialization format
- [ ] Enum serialization strategy
- [ ] Extensibility (@JsonAnySetter, additionalProperties)

### 8. Configuration

- [ ] Configuration sources (code, files, env vars)
- [ ] Required vs optional configuration
- [ ] Sensitive configuration (secrets, tokens)
- [ ] Per-request vs global configuration
- [ ] Default values

### 9. Dependencies

- [ ] Direct dependencies with versions
- [ ] Transitive dependency concerns
- [ ] Shade/relocate patterns
- [ ] Version constraints (minimum, maximum, ranges)

### 10. Test Patterns

- [ ] Testing framework (JUnit, Jest, pytest, xUnit)
- [ ] Mocking approach (Mockito, WireMock, nock, responses)
- [ ] Test fixture patterns
- [ ] Integration test setup

## Output Format

Generate `docs/discovery-report.md` with the following structure:

```markdown
# Core SDK Discovery Report

## SDK Identity

- **Name:**
- **Repository:**
- **Version Analyzed:**
- **Language:**
- **Build System:**
- **License:**

## Architecture Summary

[2-3 paragraph overview]

## Public API Surface

### [Client/Module 1]

| Method | Signature | Return Type | Notes |
| ------ | --------- | ----------- | ----- |

### [Client/Module 2]

...

## Design Patterns

[Bullet list with code examples]

## Error Handling

[Exception tree + description]

## Async/Sync Model

[Description with implications for wrapping]

## Configuration Model

[Description with examples]

## Dependencies

| Dependency | Version | Purpose | Conflict Risk |
| ---------- | ------- | ------- | ------------- |

## Key Observations

[Anything unusual, notable, or potentially problematic for wrapping]

## Monorepo Notes (if applicable)

[Package graph, dependency chain, recommended depend-on target]
```

## Completion Signal

After generating the report, say:

> **Stage 1 Complete.** I've generated the discovery report at `docs/discovery-report.md`.
> Please review it and confirm:
>
> 1. Is the API surface accurate and complete?
> 2. Are there any internal/private APIs you want to include?
> 3. Any areas where my analysis seems incorrect?
>
> Once approved, we'll proceed to **Stage 2: Target Framework Analysis**.
