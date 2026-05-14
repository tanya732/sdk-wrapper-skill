# Stage 1: Core SDK Discovery — Prompt Template

## Trigger

User provides a core SDK repository URL or package identifier.

## System Prompt

You are executing **Stage 1: Core SDK Discovery** of the SDK Wrapper Skill.

Given the core SDK repository at `{{REPO_URL}}`, perform an exhaustive analysis of its public API surface, internal architecture, and conventions.

## CRITICAL: Read, Don't Guess

You MUST read the actual source code. Do NOT hallucinate or assume APIs based on the SDK name.

**Required actions:**
1. Clone or navigate to the SDK source
2. Read the build file (pom.xml / build.gradle / package.json) for exact dependencies and versions
3. Read every public client class and extract actual method signatures
4. Read the exception/error classes for the actual hierarchy
5. Read the configuration builder/class for actual config options
6. Read the HTTP client setup for the actual networking approach

**Why this matters:** Previous iterations of this skill generated wrapper code for APIs that
didn't exist, causing compilation failures. By reading source, we ensure 100% accuracy.

## Monorepo Detection (perform first)

If the repo contains multiple modules (check `settings.gradle`, root `pom.xml` with `<modules>`, or `workspaces` in package.json):

1. List all modules and their purpose
2. Identify the **base/core module** — the one with NO framework dependencies (no Spring, Micronaut, Express, etc.)
3. Use that module as the source for discovery
4. Skip modules that are already framework wrappers (e.g., `auth0-spring-boot`)
5. If ambiguous, ask the user which module to wrap

**Example:** `auth0-auth-java` repo contains:
- `auth0` → base SDK (wrap this)
- `auth0-spring-boot` → already a Spring wrapper (skip)

## Auto-Detection (perform next)

| Property | How to detect | Record |
|----------|--------------|--------|
| Build tool | pom.xml → Maven; build.gradle → Gradle; package.json → npm | Exact tool |
| **SDK version** | gradle.properties (`VERSION_NAME`/`version`), pom.xml `<version>`, build.gradle `version = '...'`, package.json `"version"` | **Exact version string** |
| Language version | maven.compiler.source / "engines" / python_requires | Minimum version |
| Primary HTTP client | Read HTTP layer imports | Library + version |
| Serialization | Read import statements for Jackson/Gson/etc | Library + version |
| Async model | Check return types of main methods | sync/async/reactive |

### CRITICAL: SDK Version & Publish Status

The core SDK's exact version MUST be extracted and verified.

1. Check the SDK's version declaration in its build file/properties
2. **Never truncate version suffixes** — use `1.0.0-beta.1` exactly, not `1.0.0`
3. **Verify the version exists on the public registry** (Maven Central, npm, PyPI) — fetch the artifact page to confirm
4. If it IS published → use it as a direct dependency. **Do NOT assume a module isn't published just because it lives in a monorepo.** Monorepo modules are frequently published independently.
5. Only if verification CONFIRMS it's not published → configure a composite/local build and document it
6. Record the exact version and publish status in the discovery report for use in Stage 5

**Common mistake:** Seeing `auth0-api-java` inside a monorepo alongside `auth0-springboot-api` and concluding it's "not published separately." WRONG — check the registry. If it's there, depend on it directly.

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
