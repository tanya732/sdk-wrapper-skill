# Stage 5: Code Generation — Prompt Template

## Trigger

Stage 4 architecture design is complete.

## System Prompt

You are executing **Stage 5: Code Generation** of the SDK Wrapper Skill.

Using the architecture design, generate the **complete, compilable, runnable** framework SDK.
The output must work out of the box — compile with a single command, tests must pass,
and the example app must start with only a `.env` file from the user.

## Generation Order

Generate files in this order (dependencies first):

### Phase 1: Project Scaffolding

1. **Build file** (`pom.xml`, `build.gradle.kts`, `package.json`, `pyproject.toml`)
   - Include ALL dependencies with **exact versions** (not ranges)
   - Configure all plugins (compile, test, package, publish)
   - Framework dependency: `provided`/`peerDependency` scope
   - Core SDK: exact version as compile dependency
2. **Directory structure** (create all directories)
3. **CI/CD configuration** (`.github/workflows/ci.yml`)
4. **LICENSE** file (match core SDK's license)
5. **`.gitignore`** — MUST include `.env`

### Phase 2: Core Source Code

6. **Configuration class** — reads from environment variables / framework config
   - All config from env vars (never hardcoded)
   - Validates required values at startup
   - Provides clear error messages for missing config
7. **Module / Plugin class** — integrates with framework's DI/lifecycle
8. **Main client wrapper** — primary public API entry point
9. **Sub-client wrappers** (if core SDK has hierarchical clients)
10. **Error mapping layer** — core SDK errors → framework-idiomatic responses
11. **Middleware / Filters** — authentication, token validation

### Phase 3: Example Application

12. **Example app build file** (separate from SDK build)
13. **Example app source** — demonstrates ALL main features:
    - Login flow
    - Logout
    - Protected routes (require authentication)
    - Token handling (refresh, validation)
    - User info retrieval
    - Error handling display
14. **`.env.example`** — EVERY variable with comments:
    ```env
    # ===========================================
    # Auth0 Configuration
    # Get these from: Auth0 Dashboard → Applications → Your App
    # ===========================================

    # Your Auth0 tenant domain (e.g., dev-abc123.us.auth0.com)
    AUTH0_DOMAIN=your-tenant.auth0.com

    # Application Client ID
    AUTH0_CLIENT_ID=your-client-id

    # Application Client Secret (keep this safe!)
    AUTH0_CLIENT_SECRET=your-client-secret

    # Token Issuer URL
    AUTH0_ISSUER=https://your-tenant.auth0.com/

    # API Audience (if using API authorization)
    AUTH0_AUDIENCE=https://your-api-identifier

    # ===========================================
    # Application Configuration
    # ===========================================

    # Port the example app runs on
    APP_PORT=8080

    # Base URL of your application
    APP_BASE_URL=http://localhost:8080

    # OAuth callback URL (must match Auth0 dashboard settings)
    AUTH0_CALLBACK_URL=http://localhost:8080/callback

    # URL to redirect to after logout
    AUTH0_LOGOUT_URL=http://localhost:8080
    ```
15. **Example app README.md** — exactly 3 steps:
    ```markdown
    # Example App

    ## Run in 3 Steps

    ### 1. Configure
    ```bash
    cp .env.example .env
    # Edit .env with your Auth0 credentials
    ```

    ### 2. Install
    ```bash
    ../gradlew build   # or: npm install / pip install -e .
    ```

    ### 3. Run
    ```bash
    ../gradlew run     # or: npm start / python main.py
    ```

    Open http://localhost:8080
    ```

### Phase 4: Documentation

16. **README.md** with:
    - One-line description
    - Installation (one command)
    - Quick start (copy-paste-ready)
    - Full configuration reference table
    - Usage examples for main features
    - Error handling guide
    - Migration guide (from raw core SDK usage)
17. **CHANGELOG.md** — initial entry
18. **MAINTENANCE.md** — update strategy (generated in Stage 7)

## Code Quality Rules

### Must Follow

- [ ] ALL configuration comes from environment variables or framework config (NEVER hardcoded)
- [ ] Build file has exact dependency versions (not ranges like `[2.0,3.0)`)
- [ ] Every public method has documentation (Javadoc, JSDoc, docstring)
- [ ] Every error path is handled (no swallowed exceptions)
- [ ] Thread-safety addressed (singleton clients, executor for blocking in async contexts)
- [ ] Logging at appropriate levels (DEBUG for flow, WARN for issues, ERROR for failures)
- [ ] .gitignore excludes `.env`, build outputs, IDE files

### Must Avoid

- [ ] NO hardcoded credentials, domains, or client IDs anywhere
- [ ] NO copy-paste from core SDK (delegate via composition only)
- [ ] NO dependency on core SDK internals (only public API)
- [ ] NO blocking calls in async/reactive contexts without proper bridging
- [ ] NO version ranges in primary dependencies

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

## Output Verification

After generating all files, verify:

1. **Build file is complete** — All deps, all plugins, correct versions
2. **No TODO placeholders** — Every file is complete
3. **Example app is self-contained** — Has its own build file, can run independently
4. **`.env.example` lists every variable** — Nothing undocumented
5. **`.gitignore` excludes `.env`** — Secrets never committed
6. **README quick start works** — Copy-paste commands are correct

## Completion Signal

```
[Stage 5/7] Code Generation .................. Done

Generated: {N} files ({N} source + {N} config + {N} docs)
Build command: {command}
Example app: {output-dir}/example/

Proceeding to Stage 6: Test Generation...
```
