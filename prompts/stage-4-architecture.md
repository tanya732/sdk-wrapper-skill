# Stage 4: Architecture & Design — Prompt Template

## Trigger

User has approved Stage 3 feasibility report with all decisions made.

## System Prompt

You are executing **Stage 4: Architecture & Design** of the SDK Wrapper Skill.

Using the approved feature list from `docs/feasibility-report.md`, design the complete architecture for the framework SDK.

## Design Process

### 1. Package / Module Structure

Design the directory layout following the target framework's conventions:

```
{sdk-name}/
├── src/main/{lang}/com/example/{sdk}/
│   ├── {SdkName}Module.{ext}        # Framework integration entry point
│   ├── {SdkName}Config.{ext}        # Configuration binding
│   ├── client/
│   │   ├── {SdkName}Client.{ext}    # Main wrapper client
│   │   ├── Auth{Client}.{ext}       # Auth-specific wrapper (if applicable)
│   │   └── Mgmt{Client}.{ext}       # Management wrapper (if applicable)
│   ├── middleware/
│   │   ├── Auth{Middleware}.{ext}    # Authentication middleware/filter
│   │   └── RateLimit{Handler}.{ext} # Rate limiting handler
│   ├── errors/
│   │   └── {SdkName}Error.{ext}     # Framework-adapted error types
│   └── util/
│       └── {Helpers}.{ext}          # Utility classes
└── src/test/...
```

### 2. Public API Design

For each approved feature, design the wrapper method:

```
Core SDK Method:
  authApi.login(email, password).execute().getBody()

Framework SDK Method:
  auth0Play.login(email, password): Future[TokenResponse]

Design decisions:
  - Return type adaptation: Request<T> → Future[T] (Play's async model)
  - Error handling: Auth0Exception → mapped to Play's Result.unauthorized()
  - Configuration: Builder params from Play's config file (application.conf)
```

### 3. Configuration Integration Design

Map core SDK configuration to framework configuration:

```yaml
# Example: application.conf (Play/HOCON)
auth0 {
  domain = "your-tenant.auth0.com"
  domain = ${?AUTH0_DOMAIN}
  client-id = "your-client-id"
  client-id = ${?AUTH0_CLIENT_ID}
  client-secret = "your-client-secret"
  client-secret = ${?AUTH0_CLIENT_SECRET}

  # Optional
  audience = ""
  connection = ""
  timeout = 10s
}
```

Design the config → core SDK initialization mapping:

```
config.get("auth0.domain") → AuthAPI.newBuilder(domain, ...)
config.get("auth0.timeout") → Auth0HttpClient.builder().timeout(...)
```

### 4. Error Handling Design

Create a mapping table:

| Core SDK Error                     | HTTP Status | Framework Error     | User-Facing Message            |
| ---------------------------------- | ----------- | ------------------- | ------------------------------ |
| Auth0Exception                     | 500         | InternalServerError | "Authentication service error" |
| APIException (invalid_credentials) | 401         | Unauthorized        | "Invalid credentials"          |
| APIException (mfa_required)        | 403         | Forbidden           | "MFA required"                 |
| RateLimitException                 | 429         | TooManyRequests     | "Rate limited, retry after X"  |

### 5. Dependency Management Design

```xml
<!-- Core SDK dependency -->
<dependency>
  <groupId>com.auth0</groupId>
  <artifactId>auth0</artifactId>
  <version>[3.0.0, 4.0.0)</version>  <!-- semver range -->
</dependency>

<!-- Framework dependency (provided scope) -->
<dependency>
  <groupId>com.typesafe.play</groupId>
  <artifactId>play_2.13</artifactId>
  <version>${play.version}</version>
  <scope>provided</scope>
</dependency>
```

Key decisions:

- Core SDK: use semver range (minor version flexibility)
- Framework: `provided` scope (user supplies their own version)
- Avoid transitive conflicts: exclude problematic transitives if needed

### 6. Lifecycle Design

```
Application Start
  └─→ Framework DI / Module system loads {SdkName}Module
       └─→ Reads configuration from framework config
            └─→ Initializes core SDK client(s)
                 └─→ Registers singleton in DI container
                      └─→ Available for injection in controllers/handlers

Request Flow
  └─→ Controller/Handler receives request
       └─→ Injects {SdkName}Client
            └─→ Calls wrapper method
                 └─→ Delegates to core SDK
                      └─→ Maps response/error to framework types
                           └─→ Returns framework response

Application Stop
  └─→ Shutdown hook closes core SDK client
       └─→ Releases HTTP connections
```

## Output Format

Generate `docs/architecture.md`:

```markdown
# Architecture Design

## Overview

[2-3 paragraph summary of the architecture]

## Package Structure

[Directory tree with file descriptions]

## Component Diagram

[ASCII diagram showing component relationships]

## Public API

### Configuration

[Config file format with all keys, defaults, and env var overrides]

### Client API

[Method signatures with parameter and return types]

### Middleware / Filters

[List with behavior description]

### Error Types

[Error mapping table]

## Dependency Strategy

[Version ranges, scopes, conflict handling]

## Lifecycle

[Initialization → Request → Shutdown flow]

## Thread-Safety

[Concurrency model, shared state, request isolation]

## Design Decisions Log

| Decision | Options Considered | Chosen | Rationale |
| -------- | ------------------ | ------ | --------- |

## Open Questions

[Any remaining questions for user]
```

## Completion Signal

> **Stage 4 Complete.** I've generated the architecture design at `docs/architecture.md`.
> Please review:
>
> 1. Does the package structure make sense for your project?
> 2. Is the public API surface what you expected?
> 3. Any design decisions you'd like to change?
> 4. Answers to open questions (if any)
>
> Once approved, we'll proceed to **Stage 5: Code Generation**.
