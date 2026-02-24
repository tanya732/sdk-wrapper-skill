# Stage 5: Code Generation — Prompt Template

## Trigger

User has approved Stage 4 architecture design.

## System Prompt

You are executing **Stage 5: Code Generation** of the SDK Wrapper Skill.

Using the approved architecture (`docs/architecture.md`), generate the complete framework SDK source code. Every line of code must trace back to a decision in the architecture document.

## Generation Order

Generate files in this order (dependencies first):

### Phase 1: Project Scaffolding

1. **Build file** (`pom.xml`, `package.json`, `pyproject.toml`, etc.)
   - Include all dependencies with exact versions
   - Configure build plugins (compile, test, package, publish)
   - Set up CI-friendly versioning
2. **Directory structure** (create all directories)
3. **CI/CD configuration** (`.github/workflows/ci.yml`)
4. **LICENSE** file
5. **`.gitignore`** appropriate for language

### Phase 2: Core Source Code

6. **Configuration class** — reads framework config, validates, builds core SDK client
7. **Module / Plugin class** — integrates with framework's DI/lifecycle system
8. **Main client wrapper** — the primary public API entry point
9. **Sub-client wrappers** (if core SDK has hierarchical clients)
10. **Error mapping layer** — translates core SDK errors to framework errors
11. **Middleware / Filters** — authentication, rate limiting, logging

### Phase 3: Utilities & Helpers

12. **Response adapters** — convert core SDK responses to framework response types
13. **Configuration helpers** — env var loading, validation, defaults
14. **Logging integration** — bridge core SDK logging to framework logging

### Phase 4: Documentation

15. **README.md** with:
    - Installation instructions (dependency coordinates)
    - Quick start (minimum viable configuration)
    - Full configuration reference
    - Usage examples for every public method
    - Error handling guide
    - Migration guide (from raw core SDK)
16. **CHANGELOG.md** skeleton
17. **CONTRIBUTING.md**

## Code Quality Rules

### Must Follow

- [ ] Every public method has documentation (Javadoc, JSDoc, docstring, XML doc)
- [ ] Every public method validates its inputs
- [ ] Every error path is handled explicitly (no swallowed exceptions)
- [ ] Configuration keys have sensible defaults where possible
- [ ] Thread-safety is explicitly addressed (documented or enforced)
- [ ] No hard-coded values — everything configurable
- [ ] Logging at appropriate levels (DEBUG for flow, WARN for anomalies, ERROR for failures)

### Must Avoid

- [ ] No copy-paste from core SDK source (wrapper pattern only)
- [ ] No direct dependency on core SDK internals (only public API)
- [ ] No framework version-specific hacks without version guards
- [ ] No blocking calls in async contexts (or vice versa)
- [ ] No mutable shared state without synchronization

### Style Matching

- Match the core SDK's code style where possible (indentation, naming, comment style)
- Follow the target framework's conventions for framework-specific code
- When in conflict, prefer the framework's conventions (users of the wrapper are framework developers)

## README Template

```markdown
# {SDK Name}

{One-line description}: A {Framework} integration for the [{Core SDK Name}]({core-sdk-url}).

[![Maven Central](badge-url)](...) <!-- or npm, PyPI, etc. -->
[![CI](badge-url)](...)
[![License](badge-url)](...)

## Installation

{Package manager instructions}

## Quick Start

{Minimum viable example — 5-10 lines}

## Configuration

{Full configuration reference table}

| Key | Type | Default | Required | Description |
| --- | ---- | ------- | -------- | ----------- |

## Usage

### {Feature 1}

{Code example}

### {Feature 2}

{Code example}

...

## Error Handling

{Error mapping table and handling guide}

## Migration from Raw {Core SDK}

{Side-by-side comparison of raw vs wrapped usage}

## API Reference

{Link to generated API docs or inline reference}

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md).

## License

{License type} — see [LICENSE](LICENSE).
```

## Output Format

Generate all files in the project directory. After generation, provide a summary:

```markdown
## Generated Files Summary

| File                     | Purpose               | Lines |
| ------------------------ | --------------------- | ----- |
| pom.xml                  | Build configuration   | N     |
| src/main/.../Module.java | Framework integration | N     |
| ...                      | ...                   | N     |

**Total files:** N
**Total lines:** N
```

## Completion Signal

> **Stage 5 Complete.** I've generated the complete source code.
>
> **Summary:** [N files, N lines of code]
>
> Please review:
>
> 1. Does the code compile? (`mvn compile` / `npm run build` / etc.)
> 2. Does the public API look right?
> 3. Is the README accurate and helpful?
> 4. Any methods that need signature changes?
>
> Once approved, we'll proceed to **Stage 6: Test Generation**.
