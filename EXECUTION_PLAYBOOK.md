# Execution Playbook: auth0-auth-java → Play Framework SDK

This file contains the exact prompts to paste into Copilot Chat (Agent mode) to generate an auth0-play SDK. Run each stage in a **separate chat session** to avoid context window limits.

> **Prerequisite:** Make sure you're in the `sdk-wrapper-skill` workspace so Copilot picks up `.github/copilot-instructions.md` automatically.

---

## Stage 1: Core SDK Discovery

### Prompt

```
Run Stage 1 of the SDK Wrapper Skill.

Discover the core SDK at: https://github.com/auth0/auth0-auth-java

Analyze the full public API surface, design patterns, error handling, async/sync model,
configuration, dependencies, and test patterns.

Generate the output as docs/discovery-report.md in this workspace.

Use the pre-built example at docs/examples/auth0-auth-java-discovery-report.md as a reference
for format and depth — but do your own fresh analysis.
```

### What to check before approving:

- [ ] API surface is complete (AuthAPI + ManagementApi)
- [ ] Dual architecture noted (handwritten vs Fern-generated)
- [ ] Error hierarchy accurate (checked Auth0Exception vs unchecked ManagementException)
- [ ] Dependencies listed with versions
- [ ] Async/sync model documented

### Approve with:

```
Stage 1 approved. The discovery report is accurate.
```

---

## Stage 2: Target Framework Analysis

### Prompt

```
Run Stage 2 of the SDK Wrapper Skill.

The Stage 1 discovery report is at docs/discovery-report.md — read it first.

Target framework details:
- Framework: Play Framework
- Version: 2.8.x (latest 2.8 patch)
- Language: Java (not Scala)
- Minimum Java version: Java 11
- SDK name: auth0-play
- Distribution: Maven Central

Priority features (in order):
1. Login (username/password)
2. Signup
3. Token exchange (authorization code flow)
4. User info retrieval
5. Password reset
6. Token refresh
7. User management (CRUD via Management API)
8. MFA support
9. Passwordless flows
10. Organization support

Primary use cases:
1. Play Framework web app with Auth0 login (server-side rendered)
2. Play Framework API backend validating Auth0 tokens
3. Play Framework admin dashboard managing Auth0 users

Generate docs/target-analysis.md.
```

### What to check before approving:

- [ ] Play Framework lifecycle (Guice DI, application.conf) correctly described
- [ ] Play's async model (CompletionStage/CompletableFuture) noted
- [ ] Play's Action/Result/Controller patterns documented
- [ ] Play's Filter/EssentialFilter middleware model captured
- [ ] Test helpers (WithApplication, WithServer) mentioned

### Approve with:

```
Stage 2 approved. Proceed to feasibility analysis.
```

---

## Stage 3: Feasibility Analysis

### Prompt

```
Run Stage 3 of the SDK Wrapper Skill.

Read docs/discovery-report.md and docs/target-analysis.md first.

Perform a complete feature-by-feature compatibility analysis. For each auth0-java
public method, classify as Clean Fit / Adaptation Needed / Incompatible.

Run the full anomaly detection sweep (reference prompts/anomaly-detection-rules.md).

Calculate coverage percentage and generate docs/feasibility-report.md.

For any incompatible features, present me with options (Exclude / Warn / Adapt)
and wait for my decision before finalizing.
```

### Expected anomalies to look for:

- **MAJ-001:** Dual error hierarchy (checked Auth0Exception + unchecked ManagementException)
- **MAJ-002:** Builder config → Play's application.conf (HOCON)
- **MIN-003:** AuthAPI is sync, Play is async (CompletionStage) — need async bridge
- **CRIT-003:** Check Jackson version compatibility between auth0-java and Play 2.8

### Approve with:

```
Stage 3 approved. Here are my decisions for incompatible features:
[list your decisions for each ❌ feature]

Proceed to architecture design.
```

---

## Stage 4: Architecture Design

### Prompt

```
Run Stage 4 of the SDK Wrapper Skill.

Read all previous artifacts: docs/discovery-report.md, docs/target-analysis.md,
docs/feasibility-report.md.

Design the auth0-play SDK architecture:
- Package structure under com.auth0.play
- Play Module (extending play.api.inject.Module) for Guice DI
- Configuration binding from application.conf to auth0-java clients
- Auth0 Action composition for protecting routes
- Error mapping from Auth0 exceptions to Play Results
- Dependency management (auth0-java as compile, play as provided)

Generate docs/architecture.md.
```

### What to check before approving:

- [ ] Play Module wiring looks correct (bindings, eager singletons)
- [ ] application.conf schema makes sense
- [ ] Action composition pattern is idiomatic Play
- [ ] Error → Result mapping covers all cases
- [ ] Thread-safety addressed (singleton client, executor for blocking calls)

### Approve with:

```
Stage 4 approved. Generate the code.
```

---

## Stage 5: Code Generation

### Prompt

```
Run Stage 5 of the SDK Wrapper Skill.

Read docs/architecture.md for the approved design.

Generate the complete auth0-play SDK in a new directory: output/auth0-play/

Include:
1. pom.xml with auth0-java 3.0.0 dependency and play-java 2.8 as provided
2. All source code under src/main/java/com/auth0/play/
3. README.md with installation, quick start, and full usage examples
4. .github/workflows/ci.yml for GitHub Actions CI
5. CHANGELOG.md skeleton
6. CONTRIBUTING.md
7. LICENSE (MIT)

Follow the code quality rules in prompts/stage-5-code-generation.md.
```

### After generation, verify:

```bash
cd output/auth0-play && mvn compile
```

### Approve with:

```
Stage 5 approved. Code compiles. Generate tests.
```

---

## Stage 6: Test Generation

### Prompt

```
Run Stage 6 of the SDK Wrapper Skill.

Generate tests for the auth0-play SDK at output/auth0-play/.

Follow the test strategy in prompts/stage-6-test-generation.md:
1. Unit tests (mock auth0-java, test wrapper logic)
2. Configuration tests (valid, invalid, defaults, env overrides)
3. Integration tests using Play's WithApplication test helper
4. Edge case tests (null inputs, timeouts, concurrent access)

Also generate:
- Test fixtures (sample configs, mock responses)
- Mock factory for auth0-java clients
- Test helper utilities

Place everything under src/test/java/com/auth0/play/
```

### After generation, verify:

```bash
cd output/auth0-play && mvn test
```

### Approve with:

```
Stage 6 approved. All tests pass. Generate update strategy.
```

---

## Stage 7: Update Strategy

### Prompt

```
Run Stage 7 of the SDK Wrapper Skill.

Generate the update and migration strategy for auth0-play:

1. docs/update-strategy.md — versioning strategy, compatibility matrix,
   maintenance boundaries
2. MIGRATION_GUIDE.md — template for future version migrations
3. .github/workflows/check-updates.yml — weekly check for auth0-java updates
4. .github/workflows/compatibility.yml — test against different auth0-java versions

Place framework SDK files in output/auth0-play/ and strategy docs in
output/auth0-play/docs/.
```

### Approve with:

```
Stage 7 approved. SDK generation complete! 🎉
```

---

## Post-Generation Checklist

After all 7 stages:

- [ ] `cd output/auth0-play && mvn compile` — builds successfully
- [ ] `cd output/auth0-play && mvn test` — all tests pass
- [ ] README.md is accurate and complete
- [ ] Try the SDK in a sample Play app
- [ ] Fill out feedback.md with detailed notes
- [ ] Commit everything: `git add -A && git commit -m "Generated auth0-play SDK v1.0.0"`

---

## Troubleshooting

### "Copilot doesn't seem to follow the skill instructions"

- Make sure you're in the `sdk-wrapper-skill` workspace
- Check that `.github/copilot-instructions.md` exists
- Use Agent mode (not inline or panel chat)
- Explicitly say "Use the SDK Wrapper Skill" in your prompt

### "Context gets lost between stages"

- Start a fresh chat for each stage
- Always tell Copilot to "read docs/discovery-report.md first" (etc.)
- The artifacts on disk ARE the context bridge between stages

### "Generated code doesn't compile"

- Record the errors in feedback.md
- Ask Copilot to fix: "Fix the compile errors in output/auth0-play"
- This is expected for v1 — iterate!

### "Copilot skips stages"

- Be explicit: "Run ONLY Stage 3. Do not generate code yet."
- The instructions say to wait for approval, but reinforcement helps
