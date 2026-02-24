# Stage 2: Target Framework Analysis — Prompt Template

## Trigger

User has approved Stage 1 discovery report and is ready to specify the target framework.

## System Prompt

You are executing **Stage 2: Target Framework Analysis** of the SDK Wrapper Skill.

The core SDK discovery report has been approved. Now gather target framework details from the user and perform framework analysis.

## User Input Collection

Ask the user the following questions (present them all at once, let user answer in any order):

```
To design your framework SDK, I need the following information:

1. **Target Framework:** What framework are you targeting? (e.g., Play Framework, Fastify, Django)
2. **Framework Version:** What version? (e.g., Play 2.8, Fastify 4.x, Django 4.2)
3. **Language:** What language for the wrapper? (usually matches the framework)
4. **Minimum Language Version:** (e.g., Java 11+, Node 18+, Python 3.9+)
5. **Priority Features:** Which core SDK features are most important?
   - List your top 5-10, or say "all" for full coverage
6. **Use Cases:** Describe 2-3 primary use cases for the wrapper SDK
7. **Naming Preference:** What would you like to name the wrapper? (e.g., auth0-play, auth0-fastify)
8. **Distribution:** How will you publish? (Maven Central, npm, PyPI, NuGet, etc.)
```

## Framework Analysis Checklist

### 1. Lifecycle & Initialization

- [ ] Application startup / bootstrap process
- [ ] Module/plugin registration mechanism
- [ ] Shutdown/cleanup hooks
- [ ] Eager vs lazy initialization support

### 2. Dependency Injection

- [ ] DI container (Guice, Spring, InversifyJS, etc.) or manual wiring
- [ ] Registration patterns (bind, provide, register)
- [ ] Scope management (singleton, request-scoped, transient)
- [ ] Configuration injection patterns

### 3. Configuration

- [ ] Configuration file format (YAML, JSON, HOCON, .env, etc.)
- [ ] Configuration loading order / precedence
- [ ] Environment-specific overrides
- [ ] Secrets management integration
- [ ] Configuration validation approach

### 4. Request/Response Model

- [ ] Request object structure
- [ ] Response building patterns
- [ ] Content negotiation
- [ ] Streaming support
- [ ] Multipart handling

### 5. Middleware / Interceptors

- [ ] Middleware pipeline architecture
- [ ] Before/after hooks
- [ ] Error middleware
- [ ] Authentication middleware patterns
- [ ] Ordering / priority

### 6. Async Model

- [ ] Primary async paradigm (callbacks, promises, futures, coroutines)
- [ ] Event loop model
- [ ] Blocking vs non-blocking expectations
- [ ] Concurrent request handling

### 7. Error Handling

- [ ] Framework error types
- [ ] Global error handlers
- [ ] Error response formatting
- [ ] Error logging integration
- [ ] Status code mapping conventions

### 8. Testing

- [ ] Recommended test framework
- [ ] Test helpers / utilities provided
- [ ] Integration test server (FakeApplication, TestServer, etc.)
- [ ] Mocking conventions

### 9. Community Conventions

- [ ] Plugin/extension naming conventions
- [ ] Directory structure conventions
- [ ] Configuration key naming conventions
- [ ] Documentation style

## Output Format

Generate `docs/target-analysis.md`:

```markdown
# Target Framework Analysis

## Target Summary

- **Framework:**
- **Version:**
- **Language:**
- **Minimum Language Version:**
- **SDK Name:**

## User Requirements

### Priority Features

1. ...

### Primary Use Cases

1. ...

## Framework Analysis

### Lifecycle & Initialization

[Analysis]

### Dependency Injection

[Analysis]

### Configuration Model

[Analysis with examples]

### Request/Response Patterns

[Analysis with code samples]

### Middleware / Plugin System

[Analysis]

### Async Model

[Analysis — flag any mismatch with core SDK]

### Error Handling Conventions

[Analysis]

### Testing Idioms

[Analysis]

## Preliminary Compatibility Notes

[Quick assessment of obvious fits and friction points between core SDK and framework]
```

## Completion Signal

> **Stage 2 Complete.** I've generated the target analysis at `docs/target-analysis.md`.
> Please review and confirm:
>
> 1. Are your priority features correctly captured?
> 2. Is the framework analysis accurate?
> 3. Any additional requirements or constraints?
>
> Once approved, we'll proceed to **Stage 3: Feasibility Analysis & Feature Mapping**.
