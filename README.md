# SDK Wrapper Skill

An agentic Claude Code skill that generates **production-ready, framework-specific wrapper SDKs** from any existing core SDK. Give it a base SDK and a target framework — it produces a complete, runnable project where the only user input is a `.env` file with credentials.

## What It Does

This skill runs an 8-stage autonomous pipeline:

1. **Discovery** — Reads the core SDK source, extracts exact version, maps every public API
2. **Target Analysis** — Profiles the target framework's DI, config, middleware, and async model
3. **Feasibility** — Maps features 1:1, flags incompatibilities (async/sync mismatches, dependency conflicts)
4. **Architecture** — Designs package structure, public API, and internal adapters
5. **Code Generation** — Incremental: generates one file at a time, compiles after each, fixes immediately
6. **Test Generation** — Plain unit tests first, integration tests only where needed
7. **Update Strategy** — Documents maintenance, versioning, and CI/CD recommendations
8. **Build Verification** — Full clean build with self-healing loop (diagnose → fix → retry until GREEN)

## Quick Start

### Prerequisites

- [Claude Code](https://claude.ai/claude-code) CLI installed
- The skill installed at `~/.claude/skills/sdk-wrapper/`

### Usage

In any terminal with Claude Code, simply describe what you want:

```bash
claude "Generate a Micronaut wrapper SDK for https://github.com/auth0/auth0-java"
```

Or invoke the skill directly:

```bash
claude "/sdk-wrapper"
```

Then provide:
- **Core SDK**: GitHub URL, local path, or package coordinates
- **Target Framework**: Any supported framework (see below)

The skill handles everything else automatically.

## Step-by-Step: Create a Framework SDK

### Step 1: Choose Your Inputs

| Input | Example |
|-------|---------|
| Core SDK | `https://github.com/auth0/auth0-java` |
| Target Framework | Micronaut |

### Step 2: Invoke the Skill

```
Generate a Micronaut wrapper for the auth0-java SDK at https://github.com/auth0/auth0-java
```

### Step 3: Watch the Pipeline Run

```
[Stage 1/8] Discovery ........................ Done ✓
[Stage 2/8] Target Analysis .................. Done ✓
[Stage 3/8] Feasibility ...................... Done ✓
[Stage 4/8] Architecture ..................... Done ✓
[Stage 5/8] Code Generation .................. Done ✓ (compiles)
[Stage 6/8] Test Generation .................. Done ✓ (tests pass)
[Stage 7/8] Update Strategy .................. Done ✓
[Stage 8/8] Build Verification ............... Done ✓ (clean build GREEN)
```

If the skill detects critical incompatibilities (e.g., async/sync mismatch), it pauses and asks you how to proceed.

### Step 4: Run the Generated SDK

```bash
cd generated-project/example/
cp .env.example .env
# Fill in your credentials (clientId, clientSecret, domain, etc.)
./gradlew run   # or: mvn exec:java / npm start / python main.py
```

That's it. The app runs.

## What Gets Generated

```
auth0-micronaut/
├── build.gradle.kts              # Complete build config with exact dependency versions
├── settings.gradle.kts
├── src/main/java/
│   └── com/auth0/micronaut/
│       ├── Auth0Configuration.java        # @ConfigurationProperties
│       ├── Auth0Factory.java              # @Factory beans
│       ├── Auth0Filter.java               # HTTP filter for auth
│       └── Auth0ExceptionHandler.java     # Error mapping
├── src/test/java/
│   └── com/auth0/micronaut/
│       ├── Auth0ConfigurationTest.java
│       ├── Auth0FactoryTest.java
│       └── Auth0FilterTest.java
├── example/
│   ├── build.gradle.kts
│   ├── src/main/java/.../ExampleApp.java  # Working demo app
│   ├── .env.example                       # ← Only thing user fills in
│   └── README.md                          # 3-step run instructions
├── README.md
├── MAINTENANCE.md
├── CHANGELOG.md
└── LICENSE
```

## Supported Languages & Frameworks

### Core SDKs (Input — any language)

| Language | Build Tools Auto-Detected |
|----------|--------------------------|
| Java | Maven (`pom.xml`) or Gradle (`build.gradle`) |
| JavaScript/TypeScript | npm, yarn, or pnpm |
| Python | Poetry (`pyproject.toml`) or setuptools |
| .NET | dotnet CLI (`.csproj`) |
| Go | Go modules (`go.mod`) |

### Target Frameworks (Output)

| Language | Frameworks |
|----------|-----------|
| Java | Micronaut, Quarkus, Spring Boot, Play Framework, Dropwizard |
| JavaScript | Express, Fastify, NestJS |
| Python | Django, FastAPI, Flask |
| .NET | ASP.NET Core |
| Go | Gin, Echo, Fiber |

## Key Features

### Auto-Detection
The skill automatically determines:
- Build tool (Maven vs Gradle vs npm vs poetry)
- Language version from build config
- Framework version (latest stable)
- Dependency conflicts and resolutions

### Anomaly Detection
Flags issues before generating broken code:
- Async/sync mismatches between core SDK and framework
- Dependency version conflicts
- Serialization library conflicts (Jackson vs Gson)
- Thread-safety gaps
- GraalVM/native-image incompatibilities

### Zero-Config Example App
Every generated wrapper includes a runnable example where:
- ALL config comes from `.env` file (zero hardcoded values)
- `.env.example` documents every variable with comments
- README has 3 copy-paste commands to run
- Demonstrates login, logout, protected routes, token handling

## Example Invocations

**Java SDK → Micronaut:**
```
Generate a Micronaut wrapper for https://github.com/auth0/auth0-java
```

**Java SDK → Quarkus:**
```
Create a Quarkus integration SDK from the auth0-java-mvc-common library
```

**JavaScript SDK → Express:**
```
Generate an Express middleware wrapper for @auth0/auth0-spa-js
```

**Python SDK → FastAPI:**
```
Create a FastAPI integration from the auth0-python SDK
```

## How It Differs from ZeroToOneSDK

| | ZeroToOneSDK (Previous) | SDK Wrapper Skill (This) |
|---|---|---|
| Approach | Single-shot generation | 8-stage pipeline with incremental verification |
| Accuracy | Low — hallucinated APIs | High — reads actual source code |
| Build files | Often incomplete | Complete with exact versions (verified on registry) |
| Runnable? | Usually not | Yes — self-healing loop guarantees GREEN build |
| Anomaly detection | None | Catches conflicts before code gen |
| Error handling | User fixes manually | Skill auto-diagnoses and fixes (up to 3 retries) |
| User effort | Hours of debugging | Just fill in `.env` |

## Project Structure

```
sdk-wrapper-skill/
├── README.md                    # This file
├── EXECUTION_PLAYBOOK.md        # Detailed stage-by-stage guide
├── feedback.md                  # Evaluation framework
├── prompts/                     # Stage prompt templates
│   ├── orchestrator.md
│   ├── stage-1-discovery.md
│   ├── stage-2-target-analysis.md
│   ├── stage-3-feasibility.md
│   ├── stage-4-architecture.md
│   ├── stage-5-code-generation.md
│   ├── stage-6-test-generation.md
│   ├── stage-7-update-strategy.md
│   ├── stage-8-build-verification.md
│   └── anomaly-detection-rules.md
└── docs/                        # Example reports and templates
    ├── discovery-report.md
    ├── stage-2-target-analysis.md
    ├── stage-3-feasibility.md
    ├── stage-4-architecture.md
    └── examples/
```

## Feedback & Iteration

After generating an SDK, test and provide feedback:

1. Build: `mvn compile` / `gradle build` / `npm run build`
2. Test: `mvn test` / `gradle test` / `npm test`
3. Run example app
4. Record what worked and what didn't in `feedback.md`

## License

MIT
