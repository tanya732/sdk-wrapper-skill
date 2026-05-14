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
│ • SDK version (verify on registry)       │
│ • Language version (from build config)   │
│ • Framework version (latest stable)      │
│ • .env loading mechanism                 │
└──────────────────────────────────────────┘
         │
         ▼
[Stage 1: Discovery] ──── Read actual source code, extract exact version
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
[Stage 5: Code Generation] ──── INCREMENTAL: generate → compile → fix → repeat
         │                       Phase 1: Build file → verify deps resolve
         │                       Phase 2: Source → compile after each file
         │                       Phase 3: Example → compile
         │                       Phase 4: Tests → run until green
         │                       Phase 5: Documentation
         │
         ▼
[Stage 6: Additional Tests] ──── (optional) Add integration/edge-case tests
         │
         ▼
[Stage 7: Update Strategy] ──── Maintenance docs
         │
         ▼
[Stage 8: Build Verification] ── Full clean build → if fails → diagnose → fix → retry
         │                        Self-healing loop (max 3 retries per error)
         │                        Only asks user if stuck
         │
         ▼
[DONE] ──── Build is GREEN. User runs: cp .env.example .env → fill in → run
```

### Key Principle: Never Proceed with a Broken Build

The old approach was "generate everything, then compile at the end." This stacks errors and makes debugging painful. The new approach compiles after EVERY file. If it breaks, you fix it immediately — when the cause is obvious (you just wrote one file) instead of hunting through 10+ files for the problem.

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

> **⚠️ Publish Status Rule:** Never assume a monorepo module isn't published independently. Always check the registry. If the artifact exists on Maven Central/npm/PyPI, depend on it directly — do NOT work around it with exclusions, composite builds, or depending on a parent module.

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
| 1 | Core SDK URL/path | Discovery report (version, APIs, deps) |
| 2 | Discovery data + target framework | Framework profile |
| 3 | Discovery + framework profile | Feature matrix + anomaly report |
| 4 | Feasibility (all decisions resolved) | Architecture design |
| 5 | Approved architecture | **Compiling source + passing tests on disk** |
| 6 | (Optional) Green build from Stage 5 | Additional integration/edge-case tests |
| 7 | Green build | MAINTENANCE.md + docs |
| 8 | All files generated | **Verified GREEN clean build (self-healing)** |

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
| Exact SDK version extracted | 1 | Must match published version (including `-beta.1`, `-RC.2`, etc.) |
| Import paths match discovery | 5 | All imports must use paths from Stage 1 — never guess package structure |
| Build file has exact versions | 5 | No version ranges for primary deps |
| .env.example is complete | 5 | Every required variable documented |
| .gitignore excludes .env | 5 | Never commit secrets |
| No critical anomalies unresolved | 3 | Block until user decides |
| **Build compiles** | 5→6 | Run build command. Fix until green. Do not proceed otherwise. |
| Tests are plain unit tests | 6 | No container test for logic testable in isolation |
| **Tests pass** | 6 | Run test command. Fix until green. Do not declare done otherwise. |
| Example app has zero hardcoded config | 7 | Everything from environment/.env |

## Verification Approach

Verification is NOT a post-generation step — it is EMBEDDED in generation. Stage 5 compiles after every file.

**The rule:** You cannot write the next file until the current file compiles. You cannot declare Stage 5 done until tests pass.

**When something fails:**
1. Read the error message — it tells you exactly what's wrong
2. The cause is always the file you just wrote (since everything before it already compiled)
3. Fix that one file, re-compile, proceed

This makes debugging trivial — you never have to hunt through 10+ files for the problem.

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
[Stage N/8] Stage Name ........................ Done
[Stage N+1/8] Next Stage Name ................. In Progress
```

At completion (after Stage 8 passes):

```
[Stage 8/8] Build Verification ............... Done ✓

════════════════════════════════════════════
SDK Generation Complete!

Output: ./{project-name}/
Build: GREEN (all tests pass)

To run:
  cd {project-name}/example
  cp .env.example .env   # Fill in your credentials
  {run command}
════════════════════════════════════════════
```
