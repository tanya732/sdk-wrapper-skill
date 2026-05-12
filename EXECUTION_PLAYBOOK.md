# Execution Playbook

This playbook shows how to use the SDK Wrapper Skill to generate a complete framework SDK.
The skill runs as a Claude Code skill — invoke it from the terminal, IDE, or web.

---

## Prerequisites

- Claude Code CLI installed (`claude` command available)
- The `sdk-wrapper` skill installed at `~/.claude/skills/sdk-wrapper/`
- Access to the core SDK (GitHub URL or local path)

---

## Quick Run (Fully Autonomous)

For the fastest path, provide both inputs upfront:

```bash
claude "Generate a Micronaut wrapper SDK for https://github.com/auth0/auth0-java. Output to ./auth0-micronaut/"
```

The skill runs all 7 stages autonomously, only pausing if critical incompatibilities are detected.

---

## Interactive Run (Stage by Stage)

### Start the Skill

```bash
claude "/sdk-wrapper"
```

The skill will ask:
1. What is the core SDK? (URL, local path, or package coordinates)
2. What is the target framework?

Everything else is auto-detected.

---

## Stage 1: Core SDK Discovery

**What happens:** The skill reads the actual source code of the core SDK and extracts:
- All public APIs with exact method signatures
- Auth flows, config mechanisms, error hierarchy
- Dependencies with versions, async model, thread-safety

**What you see:**
```
[Stage 1/7] Discovery ........................ In Progress

Analyzing https://github.com/auth0/auth0-java...
- Build system: Maven (pom.xml detected)
- Modules: auth0, auth0-api
- Main clients: AuthAPI, ManagementAPI
- Auth flows: Authorization Code, Client Credentials, PKCE
- Async model: Synchronous with CompletableFuture variants
- HTTP client: OkHttp 4.x

[Stage 1/7] Discovery ........................ Done
```

**No action required** — skill proceeds automatically.

---

## Stage 2: Target Framework Analysis

**What happens:** The skill profiles the target framework's DI, config, middleware, async model, and build conventions.

**Auto-detected:**
- Framework version (latest stable)
- Build tool (based on framework convention)
- How .env files are loaded
- Test framework to use

**What you see:**
```
[Stage 2/7] Target Analysis .................. In Progress

Profiling Micronaut 4.x...
- DI: Compile-time via @Singleton, @Factory
- Config: application.yml with @ConfigurationProperties
- Async: Reactive (Project Reactor) + blocking support
- Build: Gradle Kotlin DSL (framework convention)
- .env loading: io.github.cdimascio:dotenv-java
- Test: @MicronautTest with JUnit 5

[Stage 2/7] Target Analysis .................. Done
```

---

## Stage 3: Feasibility Assessment

**What happens:** Maps every core SDK feature against the target framework. Classifies each as Direct/Adapted/Limited/Incompatible. Runs anomaly detection.

**What you see:**
```
[Stage 3/7] Feasibility ...................... In Progress

Feature mapping: 47 public methods analyzed
- Direct: 31 (66%)
- Adapted: 12 (26%) — async bridge needed
- Limited: 3 (6%) — pagination model differs
- Incompatible: 1 (2%) — reactive streaming

Anomaly Detection:
- CRIT: None
- MAJ: Sync→Reactive bridge needed (AuthAPI is blocking)
- MIN: Naming convention (camelCase → kebab-case in config)

Coverage: 96% — Strong Fit

[Stage 3/7] Feasibility ...................... Done
```

**Pauses only if:** Critical anomalies are found. Presents options and waits for your decision.

---

## Stage 4: Architecture Design

**What happens:** Designs package structure, public API, config class, error mapping, and the example app structure.

**What you see:**
```
[Stage 4/7] Architecture ..................... In Progress

Designing com.auth0.micronaut package:
├── Auth0Configuration.java      (@ConfigurationProperties)
├── Auth0Factory.java            (@Factory, creates AuthAPI + ManagementAPI)
├── Auth0Filter.java             (@Filter, JWT validation)
├── Auth0ExceptionHandler.java   (Auth0Exception → HTTP responses)
└── example/
    └── ExampleApplication.java  (login, callback, protected route)

Config keys: auth0.domain, auth0.client-id, auth0.client-secret, auth0.issuer
All loaded from .env via environment variables.

[Stage 4/7] Architecture ..................... Done
```

---

## Stage 5: Code Generation

**What happens:** Generates complete, compilable source code + build files + example app.

**What you see:**
```
[Stage 5/7] Code Generation .................. In Progress

Generating files:
  ✓ build.gradle.kts (Micronaut 4.3.0, auth0-java 2.12.0)
  ✓ settings.gradle.kts
  ✓ src/main/java/com/auth0/micronaut/Auth0Configuration.java
  ✓ src/main/java/com/auth0/micronaut/Auth0Factory.java
  ✓ src/main/java/com/auth0/micronaut/Auth0Filter.java
  ✓ src/main/java/com/auth0/micronaut/Auth0ExceptionHandler.java
  ✓ src/main/resources/META-INF/services/...
  ✓ example/build.gradle.kts
  ✓ example/src/main/.../ExampleController.java
  ✓ example/.env.example
  ✓ example/README.md
  ✓ README.md
  ✓ CHANGELOG.md
  ✓ LICENSE
  ✓ .gitignore

Total: 15 files generated

[Stage 5/7] Code Generation .................. Done
```

---

## Stage 6: Test Generation

**What happens:** Creates unit + integration tests using @MicronautTest, with mocked core SDK.

**What you see:**
```
[Stage 6/7] Test Generation .................. In Progress

Generating test suite:
  ✓ Auth0ConfigurationTest.java (5 tests)
  ✓ Auth0FactoryTest.java (4 tests)
  ✓ Auth0FilterTest.java (8 tests)
  ✓ Auth0ExceptionHandlerTest.java (6 tests)
  ✓ Auth0IntegrationTest.java (3 tests)
  ✓ test resources: application-test.yml

Total: 26 tests across 5 files

[Stage 6/7] Test Generation .................. Done
```

---

## Stage 7: Update Strategy

**What happens:** Generates MAINTENANCE.md, CI workflows for update checking, and version compatibility matrix.

**What you see:**
```
[Stage 7/7] Update Strategy .................. In Progress

Generating maintenance documentation:
  ✓ MAINTENANCE.md
  ✓ .github/workflows/check-updates.yml (weekly core SDK check)
  ✓ .github/workflows/ci.yml (build + test on PR)

Version strategy: Independent versioning
  Wrapper 1.x → auth0-java 2.x + Micronaut 4.x

[Stage 7/7] Update Strategy .................. Done
```

---

## After Generation: Run Your SDK

### 1. Build

```bash
cd auth0-micronaut
./gradlew build
```

### 2. Run Tests

```bash
./gradlew test
```

### 3. Run the Example App

```bash
cd example
cp .env.example .env
```

Edit `.env` with your Auth0 credentials:
```env
AUTH0_DOMAIN=your-tenant.auth0.com
AUTH0_CLIENT_ID=your-client-id
AUTH0_CLIENT_SECRET=your-client-secret
AUTH0_ISSUER=https://your-tenant.auth0.com/
APP_PORT=8080
CALLBACK_URL=http://localhost:8080/callback
```

Then run:
```bash
./gradlew run
```

Open `http://localhost:8080` — you have a working Auth0 app.

---

## Troubleshooting

### "Build fails with dependency conflict"
The skill auto-detects conflicts, but if one slips through:
```bash
./gradlew dependencies --configuration runtimeClasspath | grep auth0
```
Check for version mismatches and update `build.gradle.kts`.

### "Tests fail"
Check if test config exists:
```bash
cat src/test/resources/application-test.yml
```
Ensure mock values don't conflict with real validation.

### "Example app won't start"
Verify `.env` is complete — every variable in `.env.example` must have a value.

---

## Multi-Framework Generation

To generate wrappers for multiple frameworks from the same core SDK:

```bash
# First framework — full pipeline
claude "Generate a Micronaut wrapper for https://github.com/auth0/auth0-java. Output to ./auth0-micronaut/"

# Second framework — reuses Stage 1 discovery
claude "Generate a Quarkus wrapper for https://github.com/auth0/auth0-java. Output to ./auth0-quarkus/"
```

The skill caches the discovery report and reuses it across frameworks.

---

## Customization

### Override auto-detected build tool
```
Generate a Micronaut wrapper for auth0-java. Use Maven instead of Gradle.
```

### Specify output location
```
Generate a Quarkus wrapper for auth0-java. Output to /Users/tanya/projects/auth0-quarkus/
```

### Limit scope
```
Generate a Micronaut wrapper for auth0-java, but only wrap the AuthAPI (skip ManagementAPI).
```

### Target specific version
```
Generate a Micronaut 3.x wrapper for auth0-java 2.10.0.
```
