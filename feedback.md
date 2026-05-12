# SDK Generation Feedback

Use this template after each SDK generation to track quality and improve the skill.

## Session Details

- **Core SDK:** [e.g., auth0-java v2.12.0]
- **Target Framework:** [e.g., Micronaut 4.3.0]
- **Language:** [e.g., Java 17]
- **Build Tool Auto-Detected:** [e.g., Gradle — correct? Y/N]
- **Date:** [YYYY-MM-DD]
- **Invocation:** [e.g., `claude "Generate a Micronaut wrapper for auth0-java"`]

---

## Quick Checks (Pass/Fail)

| Check | Pass? | Notes |
|-------|-------|-------|
| Project compiles (`./gradlew build`) | | |
| Tests pass (`./gradlew test`) | | |
| Example app starts (after .env setup) | | |
| .env.example has ALL required variables | | |
| .gitignore excludes .env | | |
| No hardcoded credentials anywhere | | |
| README quick-start steps are correct | | |
| Build file has exact versions (no ranges) | | |
| Auto-detected build tool was correct | | |

---

## Stage-by-Stage Feedback

### Stage 1: Core SDK Discovery

**Did it READ actual source or guess?** [Read / Guessed]
**API surface accuracy:** [1-5] /5

**APIs it got wrong (hallucinated or missing):**
-

---

### Stage 2: Target Framework Analysis

**Framework version correct?** [Y/N]
**Build tool choice correct?** [Y/N]
**.env loading approach correct?** [Y/N]

**What it got wrong:**
-

---

### Stage 3: Feasibility Assessment

**Anomalies detected correctly?** [Y/N]
**False positives (flagged but not real):**
-
**False negatives (missed but should have flagged):**
-

---

### Stage 4: Architecture

**Package structure idiomatic?** [Y/N]
**Config approach correct?** [Y/N]

**Design issues found after building:**
-

---

### Stage 5: Code Generation

**Compiles first try?** [Y/N]
**Compile errors (if any):**
-

**Non-idiomatic code:**
-

**Missing files:**
-

---

### Stage 6: Test Generation

**Tests pass first try?** [Y/N]
**Test failures (if any):**
-

**Missing test scenarios:**
-

---

### Stage 7: Update Strategy

**CI workflows valid YAML?** [Y/N]
**Version strategy practical?** [Y/N]

---

## Example App Assessment

| Feature | Works? | Notes |
|---------|--------|-------|
| Login flow | | |
| Logout | | |
| Protected routes | | |
| Token refresh | | |
| Error pages | | |
| Missing .env error message | | |

---

## Overall Score

| Metric | Score |
|--------|-------|
| Accuracy (APIs correct) | /5 |
| Completeness (nothing missing) | /5 |
| Runnability (works out of box) | /5 |
| Idiomatic (feels framework-native) | /5 |
| **Overall** | **/20** |

---

## Improvements for Next Iteration

| Priority | Issue | Fix |
|----------|-------|-----|
| P0 (blocker) | | |
| P1 (should fix) | | |
| P2 (nice to have) | | |

---

## Comparison with Previous Attempt

| Metric | ZeroToOneSDK | This Run | Delta |
|--------|-------------|----------|-------|
| Compiles? | No | | |
| Tests pass? | No | | |
| Example runs? | No | | |
| API accuracy | ~40% | | |
| Overall | 5/20 | | |
