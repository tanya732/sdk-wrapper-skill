# Stage 7: Update & Migration Strategy — Prompt Template

## Trigger

User has approved Stage 6 test suite (ideally after confirming tests pass).

## System Prompt

You are executing **Stage 7: Update & Migration Strategy** of the SDK Wrapper Skill.

Define how the framework SDK will be maintained over time, how it tracks core SDK updates, and how users migrate between versions.

## Version Coupling Strategy

### Option A: Lockstep Versioning

```
Core SDK 3.0.0 → Framework SDK 3.0.0
Core SDK 3.1.0 → Framework SDK 3.1.0
Core SDK 4.0.0 → Framework SDK 4.0.0
```

- ✅ Simple to understand
- ❌ Framework SDK can't have its own fixes/features independent of core

### Option B: Independent Versioning with Compatibility Matrix

```
Framework SDK 1.0.0 → Core SDK [3.0.0, 3.x.x)
Framework SDK 1.1.0 → Core SDK [3.0.0, 3.x.x)  ← framework SDK fix
Framework SDK 2.0.0 → Core SDK [4.0.0, 4.x.x)  ← core SDK major bump
```

- ✅ Framework SDK can release independently
- ❌ Users must check compatibility matrix
- **Recommended for most projects**

### Option C: Composite Versioning

```
Framework SDK 3.0.0-play-1.0 (core v3.0.0, wrapper v1.0)
```

- ✅ Both versions visible
- ❌ Non-standard, tooling may not support

**Ask user:** Which versioning strategy do you prefer?

## Update Detection Guidance

### Automated Checks

```yaml
# .github/workflows/check-updates.yml
name: Check Core SDK Updates
on:
  schedule:
    - cron: "0 9 * * 1" # Weekly on Monday
  workflow_dispatch:

jobs:
  check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Check for core SDK updates
        run: |
          # Language-specific check
          # Maven: mvn versions:display-dependency-updates
          # npm: npm outdated {core-sdk}
          # pip: pip list --outdated
      - name: Create issue if update available
        if: steps.check.outputs.update_available == 'true'
        uses: actions/github-script@v7
        with:
          script: |
            github.rest.issues.create({
              owner: context.repo.owner,
              repo: context.repo.repo,
              title: 'Core SDK update available: v${newVersion}',
              body: 'Review changelog and run compatibility tests.',
              labels: ['dependency-update']
            })
```

### Compatibility Testing on Update

```yaml
# .github/workflows/compatibility.yml
name: Core SDK Compatibility
on:
  workflow_dispatch:
    inputs:
      core_sdk_version:
        description: "Core SDK version to test against"
        required: true

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        core-version: ["${{ inputs.core_sdk_version }}"]
        framework-version: ["current", "latest"]
    steps:
      - uses: actions/checkout@v4
      - name: Override core SDK version
        run: |
          # Update dependency to target version
      - name: Build
        run: # build command
      - name: Test
        run: # test command
```

## Breaking Change Detection

### What Constitutes a Breaking Change

| Change in Core SDK         | Impact on Framework SDK | Action                                        |
| -------------------------- | ----------------------- | --------------------------------------------- |
| Method removed             | ❌ Compile error        | Major version bump, migration guide           |
| Method signature changed   | ❌ Compile error        | Major version bump, migration guide           |
| New required parameter     | ⚠️ May break            | Minor/major depending on default availability |
| Return type changed        | ❌ Compile error        | Major version bump                            |
| Exception type changed     | ⚠️ Error mapping broken | Patch fix                                     |
| New method added           | ✅ No impact            | Minor version bump to add wrapper             |
| Config key renamed         | ⚠️ Runtime error        | Patch fix with deprecation                    |
| Behavior change (same API) | ⚠️ Silent breakage      | Patch fix, test update                        |

### Detection Approach

1. Pin core SDK version in CI
2. On update, diff the public API surface
3. Classify changes using table above
4. Generate migration guide for breaking changes

## Migration Guide Template

Generate `MIGRATION_GUIDE.md`:

````markdown
# Migration Guide

## Upgrading from {Framework SDK} vX.x to vY.x

### Breaking Changes

#### 1. {Change Title}

**What changed:** {Description}
**Why:** {Core SDK updated / Framework convention changed / Bug fix}

**Before (vX.x):**

```{lang}
// old code
```
````

**After (vY.x):**

```{lang}
// new code
```

**Migration steps:**

1. ...
2. ...

#### 2. {Change Title}

...

### New Features

- {Feature}: {Description}

### Deprecations

- `{method/class}`: Deprecated in favor of `{replacement}`. Will be removed in vZ.x.

### Bug Fixes

- {Fix description} ({issue link})

```

## Maintenance Boundaries

Define clearly what the framework SDK owns vs delegates:

```

┌─────────────────────────────────────────────┐
│ Framework SDK Owns (maintains & releases) │
├─────────────────────────────────────────────┤
│ • Framework integration (module/plugin) │
│ • Configuration binding │
│ • Error mapping │
│ • Middleware / filters │
│ • Response adaptation │
│ • Framework-idiomatic public API │
│ • Documentation & examples │
│ • Test suite │
└─────────────────────────────────────────────┘

┌─────────────────────────────────────────────┐
│ Core SDK Owns (framework SDK delegates) │
├─────────────────────────────────────────────┤
│ • HTTP communication │
│ • Authentication protocol implementation │
│ • Token management │
│ • API request/response models │
│ • Rate limiting logic │
│ • Retry logic │
│ • Core business logic │
└─────────────────────────────────────────────┘

````

**Rule:** Never re-implement core SDK logic. If a behavior needs to change, contribute upstream.

## Output Format

Generate `docs/update-strategy.md`:

```markdown
# Update & Migration Strategy

## Versioning
[Chosen strategy with examples]

## Compatibility Matrix
| Framework SDK | Core SDK | Framework | Language |
|--------------|----------|-----------|----------|

## Update Process
[Step-by-step for maintainers]

## Breaking Change Policy
[Classification table and response plan]

## Maintenance Boundaries
[Ownership table]

## CI/CD for Updates
[Workflow descriptions]
````

Also generate:

- `MIGRATION_GUIDE.md` (template for future use)
- `.github/workflows/check-updates.yml`
- `.github/workflows/compatibility.yml`

## Completion Signal

> **Stage 7 Complete.** All stages finished! 🎉
>
> I've generated:
>
> - `docs/update-strategy.md`
> - `MIGRATION_GUIDE.md`
> - `.github/workflows/check-updates.yml`
> - `.github/workflows/compatibility.yml`
>
> **Full SDK Generation Summary:**
> | Stage | Artifact | Status |
> |-------|----------|--------|
> | 1. Discovery | `docs/discovery-report.md` | ✅ |
> | 2. Target Analysis | `docs/target-analysis.md` | ✅ |
> | 3. Feasibility | `docs/feasibility-report.md` | ✅ |
> | 4. Architecture | `docs/architecture.md` | ✅ |
> | 5. Code | Source tree | ✅ |
> | 6. Tests | Test suite | ✅ |
> | 7. Update Strategy | Strategy + CI | ✅ |
>
> **Next steps:**
>
> 1. Build the project and verify all tests pass
> 2. Try the SDK in a sample application
> 3. Record feedback in `feedback.md`
> 4. Iterate on any issues found
