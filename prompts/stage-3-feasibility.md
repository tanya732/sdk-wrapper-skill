# Stage 3: Feasibility Analysis & Feature Mapping — Prompt Template

## Trigger

User has approved Stage 2 target analysis.

## System Prompt

You are executing **Stage 3: Feasibility Analysis & Feature Mapping** of the SDK Wrapper Skill.

Using the approved discovery report (`docs/discovery-report.md`) and target analysis (`docs/target-analysis.md`), perform a detailed feature-by-feature compatibility analysis.

## Feature Mapping Process

For **every** public method/feature in the core SDK:

### Classification

| Symbol | Classification    | Meaning                                               |
| ------ | ----------------- | ----------------------------------------------------- |
| ✅     | Clean Fit         | Direct 1:1 mapping to framework pattern exists        |
| ⚠️     | Adaptation Needed | Requires wrapper logic, conversion, or bridging       |
| ❌     | Incompatible      | Cannot be cleanly implemented in the target framework |

### For each feature classified as ⚠️ Adaptation Needed:

- Describe the specific adaptation required
- Estimate complexity: Low / Medium / High
- Note any trade-offs

### For each feature classified as ❌ Incompatible:

Present **three options** to the user and ask them to choose:

1. **Exclude:** Remove entirely. Document why in README.
2. **Warn:** Implement with runtime warnings about limitations.
3. **Adapt:** Create a custom adapter that approximates the behavior (describe the approach).

**IMPORTANT:** Do NOT proceed past this stage until the user has made a decision for every ❌ feature.

## Anomaly Detection

Run the full anomaly detection sweep:

### Critical Anomalies (must resolve before proceeding)

```
Check: Core SDK async model vs Framework async model
  - Core uses CompletableFuture → Framework expects sync calls?
  - Core uses callbacks → Framework expects Promises?
  - Core uses reactive streams → Framework expects imperative?

Check: Runtime requirements
  - Core requires Java 17+ → Target framework supports Java 11?
  - Core requires Node 20+ → Target framework supports Node 16?

Check: Dependency conflicts
  - Core uses Jackson 2.15 → Framework bundles Jackson 2.13?
  - Core uses OkHttp 4.x → Framework uses different HTTP client?
```

### Major Anomalies (warn user)

```
Check: Error model compatibility
  - Core uses checked exceptions → Framework uses unchecked?
  - Core uses Result types → Framework uses exceptions?
  - Core errors contain metadata → Framework errors are simple strings?

Check: Configuration model
  - Core uses Builder pattern → Framework uses config files?
  - Core uses env vars → Framework uses DI?

Check: Thread-safety
  - Core SDK client is thread-safe → Framework creates per-request instances?
  - Core SDK client is NOT thread-safe → Framework shares across requests?
```

### Minor Anomalies (document)

```
Check: Naming conventions
  - Core uses camelCase → Framework convention is snake_case?
  - Core uses get/set prefixes → Framework uses bare names?

Check: Optional features
  - Core has feature X → No framework equivalent, but not essential
```

## Coverage Calculation

```
Total Features: [count]
Clean Fit (✅): [count] ([percentage]%)
Adaptation (⚠️): [count] ([percentage]%)
Incompatible (❌): [count] ([percentage]%)

Coverage Score: [Clean + Adaptation] / Total = [percentage]%
```

If coverage < 60%, recommend the user reconsider the framework choice.
If coverage 60-80%, flag it as "feasible with significant effort."
If coverage > 80%, classify as "strong fit."

## Output Format

Generate `docs/feasibility-report.md`:

```markdown
# Feasibility Analysis Report

## Coverage Summary

- **Total Features:** N
- **Clean Fit (✅):** N (X%)
- **Adaptation Needed (⚠️):** N (X%)
- **Incompatible (❌):** N (X%)
- **Overall Coverage:** X%
- **Assessment:** [Strong Fit / Feasible with Effort / Reconsider]

## Feature Mapping Matrix

### Authentication API

| Feature | Core Method | Classification | Notes | Decision |
| ------- | ----------- | -------------- | ----- | -------- |

### Management API

| Feature | Core Method | Classification | Notes | Decision |
| ------- | ----------- | -------------- | ----- | -------- |

## Anomaly Report

### Critical

[List with details and resolution]

### Major

[List with details and recommendation]

### Minor

[List for documentation]

## User Decisions

| Feature | Decision | Rationale |
| ------- | -------- | --------- |

## Approved Feature List

[Final confirmed feature list for code generation]

## Risk Assessment

[Summary of risks and mitigations]
```

## Completion Signal

> **Stage 3 Complete.** I've generated the feasibility report at `docs/feasibility-report.md`.
>
> **Summary:** [X]% coverage — [assessment].
> **Anomalies found:** [N critical, N major, N minor]
> **Decisions needed:** [N features require your input]
>
> Please review and:
>
> 1. Make decisions on all ❌ features
> 2. Confirm the approved feature list
> 3. Accept or override anomaly resolutions
>
> Once approved, we'll proceed to **Stage 4: Architecture & Design**.
