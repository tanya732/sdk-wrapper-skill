# SDK Wrapper Skill — Orchestrator

This document defines the routing and execution logic for the SDK Wrapper Skill.
The skill runs as a Claude Code skill invoked via `/sdk-wrapper` or natural language.

## Entry Points

### Full Workflow (Autonomous)

**Trigger phrases:**
- "Generate a {framework} wrapper for {core SDK}"
- "Create a {framework} SDK from {core SDK}"
- "/sdk-wrapper"

**Action:** Gather core SDK + target framework, then run all 7 stages autonomously.
Only pause if critical anomalies are detected in Stage 3.

### Single Stage

**Trigger phrases:**
- "Just analyze the feasibility of wrapping {SDK} for {framework}"
- "Run only the discovery stage for {SDK}"

**Action:** Execute the specified stage. Warn if prerequisites are missing.

## Execution Flow

```
[User provides Core SDK + Target Framework]
         │
         ▼
┌─── Auto-Detection ───────────────────────┐
│ • Build tool (Maven/Gradle/npm/poetry)   │
│ • Language version (from build config)   │
│ • Framework version (latest stable)      │
│ • .env loading mechanism                 │
└──────────────────────────────────────────┘
         │
         ▼
[Stage 1: Discovery] ──── Read actual source code
         │
         ▼
[Stage 2: Target Analysis] ──── Profile framework idioms
         │
         ▼
[Stage 3: Feasibility] ──── Map features, detect anomalies
         │
    Critical anomalies? ──── YES ──→ PAUSE, present options, wait
         │ NO
         ▼
[Stage 4: Architecture] ──── Design package structure + example app
         │
         ▼
[Stage 5: Code Generation] ──── Complete, compilable project
         │
         ▼
[Stage 6: Test Generation] ──── Tests that pass
         │
         ▼
[Stage 7: Update Strategy] ──── Maintenance docs + CI
         │
         ▼
[DONE] ──── User runs: cp .env.example .env → fill in → run
```

## Auto-Detection Rules

Before starting Stage 1, determine these automatically:

| What | How to detect | Fallback |
|------|--------------|----------|
| Build tool | Check for pom.xml / build.gradle / package.json / pyproject.toml | Ask user |
| **Core SDK version** | gradle.properties `VERSION_NAME`/`version`, pom.xml `<version>`, package.json `"version"` | Verify on Maven Central/npm/PyPI |
| Language version | Read build config (maven.compiler.source, engines, python_requires) | Use framework minimum |
| Framework version | Use latest stable release | Ask user if ambiguous |
| .env loading | Match framework convention (see table below) | dotenv library |
| Output directory | User specifies, or `./{sdk-name}-{framework}/` | Ask user |

> **⚠️ Version Rule:** Always use the EXACT version string (including `-beta.X`, `-SNAPSHOT`, `-RC.X` suffixes) in all generated build files. Never truncate. Verify it resolves from the public package registry before generating code.

### .env Loading by Framework

| Framework | Package | Config |
|-----------|---------|--------|
| Micronaut | `io.github.cdimascio:dotenv-java` | Loaded in Application.java |
| Quarkus | Built-in (.env auto-loaded) | No extra config |
| Spring Boot | `me.paulschwarz:spring-dotenv` | spring.config.import |
| Play | `io.github.cdimascio:dotenv-java` | Loaded in Module |
| Express | `dotenv` | require('dotenv').config() |
| Fastify | `dotenv` or `@fastify/env` | Loaded at startup |
| Django | `django-environ` | In settings.py |
| FastAPI | `pydantic-settings` | BaseSettings with .env |

## Stage Prerequisites

| Stage | Requires | Produces |
|-------|----------|----------|
| 1 | Core SDK URL/path | Discovery report (in memory) |
| 2 | Discovery data + target framework | Framework profile (in memory) |
| 3 | Discovery + framework profile | Feature matrix + anomaly report |
| 4 | Feasibility (all decisions resolved) | Architecture design |
| 5 | Approved architecture | Complete source tree on disk |
| 6 | Generated source (compilable) | Test suite on disk |
| 7 | Generated tests | MAINTENANCE.md + CI workflows |

## Pause Conditions

The skill runs autonomously EXCEPT when:

1. **Critical anomalies detected (Stage 3)** — Present options:
   - Continue with documented limitations
   - Choose different framework
   - Exclude incompatible features

2. **User explicitly requested interactive mode** — Pause after each stage for approval.

3. **Missing information** — If core SDK URL or target framework not provided.

## Output Requirements

Every generated project MUST include:

```
{output-dir}/
├── build file                 # Complete, all deps with exact versions
├── src/main/                  # Wrapper source code
├── src/test/                  # Tests that pass
├── example/
│   ├── build file             # Example app build
│   ├── src/                   # Working demo app
│   ├── .env.example           # ALL config with comments
│   └── README.md              # 3-step: copy .env, install, run
├── README.md                  # Installation + usage + config reference
├── MAINTENANCE.md             # Update strategy
├── CHANGELOG.md               # Initial entry
├── LICENSE                    # Match core SDK license
├── .gitignore                 # Excludes .env, build artifacts
└── .github/workflows/ci.yml   # Build + test on PR
```

## Quality Gates

| Gate | Stage | Rule |
|------|-------|------|
| Source code read (not guessed) | 1 | Must read actual files from SDK |
| Build tool detected | 1 | Must identify from project files |
| No critical anomalies unresolved | 3 | Block until user decides |
| Build file has exact versions | 5 | No version ranges for primary deps |
| .env.example is complete | 5 | Every required variable documented |
| .gitignore excludes .env | 5 | Never commit secrets |
| Tests reference correct frameworks | 6 | Must use framework test utilities |
| Example app has zero hardcoded config | 5 | Everything from environment/.env |

## Error Recovery

| Problem | Action |
|---------|--------|
| Can't access SDK repo | Ask for local path or credentials |
| Framework version conflict with SDK | Suggest compatible version combination |
| Build fails after generation | Show error, fix, regenerate affected files |
| Tests fail | Show failure, fix test or source, re-verify |

## Progress Reporting

At each stage transition, output:

```
[Stage N/7] Stage Name ........................ Done
[Stage N+1/7] Next Stage Name ................. In Progress
```

At completion:

```
SDK Generation Complete!

Output: ./auth0-micronaut/
Files: 15 source + 5 test + 3 config

To run:
  cd auth0-micronaut/example
  cp .env.example .env   # Fill in your credentials
  ../gradlew run         # Start the app
```
