# Stage 5: Code Generation — Prompt Template

## Trigger

Stage 4 architecture design is complete.

## System Prompt

You are executing **Stage 5: Code Generation** of the SDK Wrapper Skill.

Using the architecture design, generate the framework SDK **incrementally with continuous verification**. After each phase, compile/build to catch errors immediately — do not accumulate errors across phases.

## CRITICAL: Incremental Generation with Verification

**Do NOT generate all code at once.** Generate in phases, verifying each phase compiles before moving to the next. This catches errors (wrong imports, missing deps, API mismatches) one at a time instead of stacking 10+ errors at the end.

```
Phase 1: Build file → verify deps resolve
Phase 2: Source files → compile after each file
Phase 3: Example app → compile
Phase 4: Tests → run tests
Phase 5: Documentation
```

If any phase fails, FIX IT before moving to the next. Never proceed with a broken build.

---

## Phase 1: Build File + Scaffolding

Generate these files:
1. **Build file** (`pom.xml`, `build.gradle`, `package.json`, `pyproject.toml`)
   - ALL dependencies with exact versions (not ranges)
   - **Use the exact core SDK version from Stage 1** (e.g., `1.0.0-beta.1`, NOT `1.0.0`)
   - Configure all plugins (compile, test, package)
   - Core SDK as direct dependency (it IS published — verified in Stage 1)
2. **Directory structure** (create all source directories)
3. **`.gitignore`** — MUST include `.env`
4. **LICENSE** (match core SDK)

### ✅ CHECKPOINT: Verify dependencies resolve

```bash
# Java/Gradle
./gradlew dependencies --configuration compileClasspath

# Java/Maven
mvn dependency:resolve

# JavaScript
npm install

# Python
pip install -e ".[dev]"
```

**If this fails:** The dependency version is wrong or the artifact doesn't exist. Fix the build file. Do NOT proceed.

---

## Phase 2: Source Code (one file at a time)

Generate source files in dependency order. **Compile after EACH file** to catch import errors immediately.

**Order:**
1. **Configuration class** (config mapping / properties)
2. **Producer / Factory** (creates core SDK beans)
3. **Data classes** (principal, identity, request objects)
4. **Error mapping** (exception mapper / error handler)
5. **Main mechanism** (filter, middleware, authentication mechanism)

### ✅ CHECKPOINT: Compile after each file

```bash
# Java/Gradle
./gradlew compileJava

# Java/Maven
mvn compile

# JavaScript
npm run build   # or: npx tsc --noEmit

# Python
python -c "import your_package"
```

**If compilation fails after a file:**
1. Read the error message — it tells you exactly what's wrong
2. Common causes: wrong import path, wrong method signature, missing type
3. Fix the file that caused the error
4. Re-compile to confirm the fix
5. Only then proceed to the next file

---

## Phase 3: Example Application

Generate the example app:
1. **Example build file** (depends on the wrapper library)
2. **Example source** — minimal app demonstrating main features:
   - Public endpoint (no auth)
   - Protected endpoint (any valid token)
   - Scoped endpoint (requires specific permission)
3. **`.env.example`** — every required variable with comments
4. **Example README.md** — exactly 3 steps: configure, install, run

### ✅ CHECKPOINT: Example compiles

```bash
# Java/Gradle (multi-module)
./gradlew :example:compileJava

# JavaScript
cd example && npm install && npm run build

# Python
cd example && pip install -r requirements.txt
```

---

## Phase 4: Tests

Generate tests using **plain unit tests** (not container tests) wherever possible.

1. **Configuration tests** — verify config mapping works
2. **Producer/factory tests** — verify beans are created correctly
3. **Data mapping tests** — verify claims → roles, identity building
4. **Error mapping tests** — verify exceptions → HTTP responses

### ✅ CHECKPOINT: Tests pass

```bash
./gradlew test   # or: npm test / pytest
```

**If tests fail:**
1. Read the failure — is it a missing dep, wrong assertion, or actual bug?
2. Fix the issue (add dep, fix test, or fix source)
3. Re-run until GREEN
4. Only then proceed

---

## Phase 5: Documentation

Generate ONLY after all code compiles and tests pass:
1. **README.md** — installation, configuration reference, usage examples
2. **CHANGELOG.md** — initial entry
3. **MAINTENANCE.md** — update strategy (generated in Stage 7)

## Code Quality Rules

### Must Follow

- [ ] ALL configuration comes from environment variables or framework config (NEVER hardcoded)
- [ ] Build file has exact dependency versions (not ranges like `[2.0,3.0)`)
- [ ] Every public method has documentation (Javadoc, JSDoc, docstring)
- [ ] Every error path is handled (no swallowed exceptions)
- [ ] Thread-safety addressed (singleton clients, executor for blocking in async contexts)
- [ ] Logging at appropriate levels (DEBUG for flow, WARN for issues, ERROR for failures)
- [ ] .gitignore excludes `.env`, build outputs, IDE files
- [ ] **Import paths match EXACTLY what was discovered in Stage 1** — never guess package paths

### Must Avoid

- [ ] NO hardcoded credentials, domains, or client IDs anywhere
- [ ] NO copy-paste from core SDK (delegate via composition only)
- [ ] NO dependency on core SDK internals (only public API)
- [ ] NO blocking calls in async/reactive contexts without proper bridging
- [ ] NO version ranges in primary dependencies

### Build File Completeness

**Principle:** Every dependency that will be loaded at compile time or test runtime MUST be explicitly declared. Do not assume transitive dependencies will be available — framework BOMs often exclude or override them.

**How to verify:** After writing the build file, trace the execution path of each test class:
1. What classes does the test instantiate directly?
2. What do those classes load at construction time (static initializers, field types)?
3. Are any of those transitively-pulled libraries excluded by the framework BOM?

If in doubt, declare explicitly. A redundant dependency declaration is harmless; a missing one breaks the build.

### Build Tool Auto-Detection

Use the build tool detected in Stage 1:

| Core SDK uses | Framework prefers | Decision |
|---------------|-------------------|----------|
| Maven | Maven | Maven |
| Maven | Gradle | Gradle (framework wins) |
| Gradle | Gradle | Gradle |
| Gradle | Maven | Gradle (already compatible) |
| npm | npm | npm |
| poetry | poetry | poetry |

When framework convention differs from core SDK, **framework convention wins**
(users of the wrapper are framework developers).

### .env Loading Patterns

**Java (Micronaut):**
```java
// In Application.java or Factory
import io.github.cdimascio.dotenv.Dotenv;

Dotenv dotenv = Dotenv.configure().ignoreIfMissing().load();
// Properties are available via System.getenv() or Micronaut's @Value
```

**Java (Quarkus):**
```java
// Quarkus auto-loads .env — no code needed
// Access via @ConfigProperty(name = "AUTH0_DOMAIN")
```

**Java (Spring Boot):**
```java
// In application.properties:
// spring.config.import=optional:file:.env[.properties]
// Or use spring-dotenv dependency
```

**JavaScript (Express/Fastify):**
```javascript
import 'dotenv/config';
// process.env.AUTH0_DOMAIN is now available
```

**Python (FastAPI):**
```python
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    auth0_domain: str
    auth0_client_id: str

    class Config:
        env_file = ".env"
```

## Completion Criteria

Stage 5 is complete ONLY when ALL of these are true:

- [ ] `./gradlew compileJava` (or equivalent) passes with zero errors
- [ ] `./gradlew :example:compileJava` (or equivalent) passes
- [ ] `./gradlew test` passes (all tests green)
- [ ] `.env.example` lists every required variable
- [ ] `.gitignore` excludes `.env`
- [ ] No TODO placeholders in any file
- [ ] README quick-start commands are correct

## Completion Signal

```
[Stage 5/7] Code Generation .................. Done ✓

Verification:
  ✓ Dependencies resolve
  ✓ Source compiles ({N} files)
  ✓ Example compiles
  ✓ Tests pass ({N} tests)

Generated: {output-dir}/
Proceeding to Stage 7: Update Strategy...
```

**Note:** With incremental verification, Stage 5 now subsumes Stage 6 (tests are generated and verified as part of Phase 4). The separate Stage 6 prompt remains available for adding MORE tests later but is no longer a hard prerequisite.
