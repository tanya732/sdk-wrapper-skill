# SDK Wrapper Skill

A **GitHub Copilot Agent Skill** that auto-scaffolds framework-specific wrapper SDKs on top of any core SDK. Designed for SDK owners and maintainers who need to create framework integrations (e.g., Play Framework, Dropwizard, Fastify, Express, Django, ASP.NET) that leverage existing core SDKs.

## 🎯 What It Does

This skill acts as an interactive, multi-stage agent that:

1. **Discovers** a core SDK's complete API surface from any repository
2. **Analyzes** the target framework's patterns and conventions
3. **Assesses** feasibility by mapping core features to framework idioms
4. **Designs** the architecture with proper dependency management
5. **Generates** complete source code, tests, and documentation
6. **Plans** update strategies for ongoing maintenance

## 🚀 Quick Start

### Prerequisites

- GitHub Copilot with Agent mode enabled
- Access to the core SDK repository (public or private)
- Knowledge of your target framework

### Usage

Open Copilot Chat in VS Code and say:

```
@workspace Use the SDK Wrapper Skill to generate a framework SDK.

Core SDK: https://github.com/auth0/auth0-auth-java
Target Framework: Play Framework 2.8
Language: Java
```

The skill will guide you through each stage interactively.

## 📋 Stages

| Stage | Name                      | Output                  | Approval Required |
| ----- | ------------------------- | ----------------------- | ----------------- |
| 1     | Core SDK Discovery        | `discovery-report.md`   | ✅                |
| 2     | Target Framework Analysis | `target-analysis.md`    | ✅                |
| 3     | Feasibility Analysis      | `feasibility-report.md` | ✅                |
| 4     | Architecture & Design     | `architecture.md`       | ✅                |
| 5     | Code Generation           | Complete source tree    | ✅                |
| 6     | Test Generation           | Complete test suite     | ✅                |
| 7     | Update Strategy           | `update-strategy.md`    | ✅                |

## 🌍 Supported Languages & Frameworks

### Core SDKs (Source)

- **Java** — Maven/Gradle projects, sync and async patterns
- **JavaScript/TypeScript** — npm packages, monorepos, ESM/CJS
- **Python** — pip packages, async/sync
- **.NET** — NuGet packages, async/await
- **Go** — Go modules
- **Any language** with a public Git repository

### Target Frameworks (Examples)

- **Java:** Play Framework, Dropwizard, Spring Boot, Micronaut, Quarkus
- **JavaScript:** Fastify, Express, Next.js, NestJS, Hono
- **Python:** Django, Flask, FastAPI, Starlette
- **.NET:** ASP.NET Core, Minimal APIs
- **Go:** Gin, Echo, Fiber

## 🔍 Key Features

### Anomaly Detection

Automatically detects and surfaces:

- Async/sync mismatches between core SDK and target framework
- Dependency conflicts
- Configuration model incompatibilities
- Thread-safety concerns
- Serialization differences

### Feasibility Analysis

For each core SDK feature, classifies as:

- ✅ **Clean Fit** — Direct mapping to framework patterns
- ⚠️ **Adaptation Needed** — Requires wrapper logic
- ❌ **Incompatible** — Cannot be cleanly implemented

For incompatible features, presents options:

1. **Exclude** entirely with documentation
2. **Implement with warnings** about limitations
3. **Create custom adapter** solutions

### Complete Package Generation

- Source code with framework-idiomatic patterns
- Comprehensive test suite (unit + integration)
- README with quick start and examples
- Build configuration (Maven, npm, pip, etc.)
- CI/CD pipeline configuration
- Migration guide templates
- Changelog skeleton

### Update Strategy

- Version coupling between core and framework SDK
- Breaking change detection guidance
- Migration guide generation
- Maintenance boundary definitions

## 🏗 Architecture

```
┌──────────────────────────────────────────────────┐
│                  User (SDK Owner)                 │
├──────────────────────────────────────────────────┤
│              Copilot Agent Skill                  │
│  ┌────────────┐  ┌─────────────┐  ┌───────────┐ │
│  │  Discovery  │→│  Analysis   │→│  Design    │ │
│  │  Engine     │  │  Engine     │  │  Engine    │ │
│  └────────────┘  └─────────────┘  └───────────┘ │
│  ┌────────────┐  ┌─────────────┐  ┌───────────┐ │
│  │  Code Gen   │→│  Test Gen   │→│  Update    │ │
│  │  Engine     │  │  Engine     │  │  Strategy  │ │
│  └────────────┘  └─────────────┘  └───────────┘ │
├──────────────────────────────────────────────────┤
│          Anomaly Detection (Cross-cutting)        │
├──────────────────────────────────────────────────┤
│  Core SDK Repo ←──── GitHub API ────→ Framework  │
│                                        Docs       │
└──────────────────────────────────────────────────┘
```

## 📁 Generated SDK Structure

```
auth0-{framework}/
├── README.md
├── CHANGELOG.md
├── CONTRIBUTING.md
├── MIGRATION_GUIDE.md
├── LICENSE
├── pom.xml / package.json / pyproject.toml
├── src/
│   ├── main/
│   │   └── {language-specific source tree}
│   └── test/
│       └── {language-specific test tree}
├── docs/
│   ├── architecture.md
│   ├── discovery-report.md
│   ├── feasibility-report.md
│   ├── target-analysis.md
│   └── update-strategy.md
└── .github/
    └── workflows/
        └── ci.yml
```

## 🧪 Testing the Skill

After generating an SDK, test it by:

1. Building the project: `mvn compile` / `npm run build` / etc.
2. Running tests: `mvn test` / `npm test` / etc.
3. Recording feedback in `feedback.md` for iteration

## 📄 Feedback Template

Create `feedback.md` to track where the skill fell short:

```markdown
# SDK Generation Feedback

## Core SDK: [name]

## Target Framework: [name]

## Date: [date]

### What Worked Well

-

### What Fell Short

-

### Missing Features

-

### Incorrect Assumptions

-

### Suggestions for Improvement

-
```

## 🤝 Contributing

This skill is iteratively developed. After testing:

1. Record detailed feedback in `feedback.md`
2. Identify pattern gaps
3. Update the skill instructions
4. Re-test with the same or different SDK/framework combination

## 🧑‍💻 Example: Generating a Wrapper SDK

This example shows how an SDK owner or developer can use the SDK Wrapper Skill to generate a Play Framework wrapper SDK for the core SDK `auth0-api-java` (from the `auth0-auth-java` monorepo).

### 1. Start the Skill in Copilot Chat

Open Copilot Chat in VS Code and enter:

```
@workspace Use the SDK Wrapper Skill to generate a framework SDK.

Core SDK: https://github.com/auth0/auth0-auth-java
Target Framework: Play Framework 2.8
Language: Java
Key Use Case: JWT validation middleware
```

The skill will guide you through each stage interactively, prompting for approval before proceeding.

### 2. Stage-by-Stage Workflow

**Stage 1: Core SDK Discovery**

- The skill analyzes the core SDK's API surface, patterns, and conventions.
- Output: `discovery-report.md`
- **Prompt Example:**
  > Run Stage 1: Discover the core SDK at https://github.com/auth0/auth0-auth-java

**Stage 2: Target Framework Analysis**

- The skill asks for framework details and analyzes Play's lifecycle, DI, and middleware patterns.
- Output: `target-analysis.md`
- **Prompt Example:**
  > Run Stage 2: Target framework is Play Framework 2.8, Java 11+, wrapper should provide JWT validation as Play action/middleware.

**Stage 3: Feasibility Analysis & Feature Mapping**

- The skill maps core SDK features to Play idioms, detects anomalies, and presents options for incompatible features.
- Output: `feasibility-report.md`
- **Prompt Example:**
  > Run Stage 3: Map core SDK JWT validation to Play middleware. Highlight any incompatibilities.

**Stage 4: Architecture & Design**

- The skill proposes the wrapper SDK's structure, DI integration, and error mapping.
- Output: `architecture.md`
- **Prompt Example:**
  > Run Stage 4: Design Play module structure for JWT validation using auth0-api-java.

**Stage 5: Code Generation**

- The skill generates the full source tree, including:
  - `pom.xml` or `build.sbt`
  - `JwtValidationAction.java` (Play action using core SDK)
  - DI wiring
  - README, docs, CI config
- Output: Complete source tree
- **Prompt Example:**
  > Run Stage 5: Generate Play module code for JWT validation.

**Stage 6: Test Generation**

- The skill generates unit and integration tests for the wrapper SDK.
- Output: Complete test suite
- **Prompt Example:**
  > Run Stage 6: Generate tests for JWT validation action.

**Stage 7: Update & Migration Strategy**

- The skill generates update and migration documentation.
- Output: `update-strategy.md`, `MIGRATION_GUIDE.md`
- **Prompt Example:**
  > Run Stage 7: Generate update and migration docs.

### 3. Example Output: Play JWT Validation Action

The generated code will include a Play action that validates JWTs using the core SDK:

```java
// src/main/java/com/example/play/JwtValidationAction.java
package com.example.play;

import play.mvc.*;
import java.util.concurrent.CompletionStage;
import com.auth0.jwt.interfaces.DecodedJWT;
import com.auth0.jwt.exceptions.JWTVerificationException;
import com.auth0.jwt.JWTVerifier;

public class JwtValidationAction extends Action.Simple {
    private final JWTVerifier verifier;

    public JwtValidationAction(JWTVerifier verifier) {
        this.verifier = verifier;
    }

    @Override
    public CompletionStage<Result> call(Http.Context ctx) {
        String authHeader = ctx.request().getHeader("Authorization");
        if (authHeader == null || !authHeader.startsWith("Bearer ")) {
            return CompletableFuture.completedFuture(Results.unauthorized("Missing token"));
        }
        String token = authHeader.substring("Bearer ".length());
        try {
            DecodedJWT jwt = verifier.verify(token);
            ctx.args.put("jwt", jwt);
            return delegate.call(ctx);
        } catch (JWTVerificationException e) {
            return CompletableFuture.completedFuture(Results.unauthorized("Invalid token"));
        }
    }
}
```

### 4. Approve Each Stage

After each stage, review the generated artifact (report, code, or doc) and approve to continue. You can request changes or stop at any stage.

### 5. Build, Test, and Iterate

- Build the generated SDK: `mvn compile` or `sbt compile`
- Run tests: `mvn test` or `sbt test`
- Integrate the wrapper into your Play app
- Provide feedback or iterate as needed

---

This workflow ensures you generate a framework-idiomatic, anomaly-aware wrapper SDK with full documentation and test coverage, tailored to your use case and the core SDK's actual capabilities.

## 🟢 How to Use This Skill in Your Own Repo (e.g., auth0-auth-java)

1. **Open your target repo in VS Code.**
   - Open the `auth0-auth-java` repository as your workspace.

2. **Open Copilot Chat (or Copilot Agent) in VS Code.**

3. **Start the Skill with a Natural Language Command:**
   - Type this in Copilot Chat:

     ```
     @workspace Use the SDK Wrapper Skill to generate a framework SDK.

     Core SDK: https://github.com/auth0/auth0-auth-java
     Target Framework: Play Framework 2.8
     Language: Java
     Key Use Case: JWT validation middleware
     ```

   - Change the target framework, language, or use case as needed for your project.

4. **Follow the Interactive Prompts:**
   - The skill will guide you through each stage (discovery, analysis, design, code generation, etc.).
   - After each stage, review the generated file or code and approve to continue.

5. **Review and Approve Each Step:**
   - You’re always in control. You can stop, ask for changes, or continue at each stage.

6. **Build and Test the Generated SDK:**
   - Use standard build commands (`mvn compile`, `mvn test`, etc.) in your repo.

7. **Integrate the Wrapper SDK into Your App:**
   - Use the generated Play module or middleware in your Play Framework app.

---

**In Short:**

- Open your repo in VS Code
- Open Copilot Chat
- Tell Copilot what you want (see the example command above)
- Approve each step as Copilot generates code and docs
- Build, test, and use your new wrapper SDK!

## 📜 License

MIT
