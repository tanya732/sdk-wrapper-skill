# Stage 8: Build Verification & Self-Healing — Prompt Template

## Trigger

Stage 7 (Update Strategy) is complete. All source files, tests, example app, and documentation have been generated.

## System Prompt

You are executing **Stage 8: Build Verification & Self-Healing** of the SDK Wrapper Skill.

This is the final gate. Run a full clean build of the entire project (library + tests + example). If anything fails, diagnose, fix, and re-run — automatically — until the build is fully GREEN. Do not ask the user for help unless you are stuck after 3 attempts on the same error.

## Execution Loop

```
┌─────────────────────────────────────┐
│  1. Clean build (library)           │
│  2. Run tests                       │
│  3. Build example app               │
│                                     │
│  ALL GREEN? ──── YES ──→ DONE ✓     │
│       │                             │
│       NO                            │
│       │                             │
│  4. Read error output               │
│  5. Diagnose root cause             │
│  6. Apply fix                       │
│  7. Go to step 1                    │
│                                     │
│  3 attempts on same error? ──→ ASK  │
└─────────────────────────────────────┘
```

## Step 1: Full Clean Build

Run the complete build from scratch (no cached artifacts):

```bash
# Java/Gradle
./gradlew clean build

# Java/Maven
mvn clean verify

# JavaScript
rm -rf node_modules && npm install && npm run build && npm test

# Python
rm -rf .venv && python -m venv .venv && pip install -e ".[dev]" && pytest
```

## Step 2: Diagnose Failures

When a build fails, classify the error:

| Error Type | Symptom | Fix Strategy |
|------------|---------|--------------|
| **Dependency not found** | "Could not find artifact X:Y:Z" | Wrong version or artifact ID. Check registry. Fix build file. |
| **Import does not exist** | "cannot find symbol" / "Module not found" | Wrong package path. Check Stage 1 discovery. Fix import. |
| **Method does not exist** | "cannot find method X" / "has no attribute X" | API mismatch. Re-read the core SDK source. Fix the call. |
| **Type mismatch** | "incompatible types" / "expected X got Y" | Wrong return type or parameter. Fix signature. |
| **Missing transitive dep** | "ClassNotFoundException" / "NoClassDefFoundError" | A runtime dep isn't on classpath. Add explicit dependency. |
| **Test assertion failure** | "expected X but was Y" | Either the test expectation is wrong, or the source logic is wrong. Read both. |
| **Configuration error** | "Missing required config" / "Unknown property" | Config key name mismatch. Align config class with properties file. |

## Step 3: Apply Fix

Rules for fixing:

1. **Fix the root cause, not the symptom.** If a test fails because the source is wrong, fix the source — don't weaken the test.
2. **One fix at a time.** Apply the fix, then re-run the full build. Don't batch multiple guesses.
3. **Stay consistent with Stage 1 discovery.** When in doubt about an API, re-read the core SDK source — don't guess.
4. **Never suppress errors.** Don't add `@SuppressWarnings`, `// @ts-ignore`, `type: ignore`, or try/catch blocks just to make the build pass.

## Step 4: Re-run

After each fix, re-run the FULL clean build (step 1). Do not skip the clean — cached artifacts can mask issues.

## Escalation

If you hit the same error 3 times without resolving it:

1. Present the error to the user
2. Show what you've tried
3. Ask for guidance

Common situations requiring escalation:
- The core SDK has a bug or undocumented behavior
- The framework version has a known incompatibility
- A transitive dependency conflict that can't be resolved without shading

## Success Criteria

Stage 8 is complete when ALL of these pass in a single clean run:

- [ ] Library compiles with zero errors and zero warnings (or only framework-standard warnings)
- [ ] All tests pass (zero failures, zero skipped unless intentionally `@Disabled` with reason)
- [ ] Example app compiles
- [ ] No TODO/FIXME markers left in source code

## Output

```
[Stage 8/8] Build Verification ............... Done ✓

Full clean build: PASSED
  ✓ Library compiles (0 errors)
  ✓ Tests pass (N/N green)
  ✓ Example compiles

Iterations needed: {N}
Fixes applied: {list of fixes, if any}

════════════════════════════════════════════
SDK Generation Complete!

Output: {output-dir}/
To run:
  cd {output-dir}/example
  cp .env.example .env   # Fill in your credentials
  {run command}
════════════════════════════════════════════
```

## If Zero Fixes Needed

If the build passes on first try (thanks to incremental verification in Stage 5), simply report:

```
[Stage 8/8] Build Verification ............... Done ✓ (first attempt)
```

This stage serves as the final safety net — even if Stage 5's incremental checks caught most issues, a clean rebuild from scratch may surface issues that incremental builds miss (stale caches, ordering problems, resource files).
