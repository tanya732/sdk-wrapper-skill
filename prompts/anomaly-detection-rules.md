# Anomaly Detection Reference

This document defines the complete set of anomaly detection rules the SDK Wrapper Skill checks during Stage 3 (Feasibility Analysis). Each rule has a severity, detection method, and recommended resolution.

---

## Critical Anomalies

> **Must resolve before proceeding to Stage 4.** These indicate fundamental incompatibilities.

### CRIT-001: Async/Sync Paradigm Mismatch

**Detect:** Core SDK uses `CompletableFuture`, `Promise`, `async/await`, `Task`, or reactive types (`Flux`, `Observable`) as primary return types, BUT target framework expects synchronous responses (or vice versa).

**Examples:**

- Core SDK returns `CompletableFuture<User>` → Target framework controller expects `User` synchronously
- Core SDK is synchronous → Target framework is fully reactive (WebFlux, Vert.x)

**Resolution Options:**

1. **Bridge:** Use `.get()` / `await` / `.block()` to adapt (with timeout)
2. **Native async:** If framework supports both, use async path
3. **Reject:** If paradigm mismatch is total (e.g., sync-only SDK in reactive-only framework), recommend different approach

**Risk if ignored:** Deadlocks, thread starvation, or blocked event loops.

---

### CRIT-002: Runtime Version Incompatibility

**Detect:** Core SDK requires a language/runtime version higher than what the target framework supports.

**Examples:**

- Core SDK requires Java 17+ (records, sealed classes) → Framework only supports Java 11
- Core SDK requires Node 20+ (built-in fetch) → Framework supports Node 16+

**Resolution Options:**

1. **Upgrade framework** to support required runtime
2. **Use older core SDK version** compatible with the runtime
3. **Reject** if no compatible combination exists

**Risk if ignored:** Compile errors, runtime crashes.

---

### CRIT-003: Hard Dependency Conflict

**Detect:** Core SDK and target framework both depend on the same library but require incompatible versions.

**Examples:**

- Core SDK requires Jackson 2.15+ → Framework bundles Jackson 2.12 (binary incompatible)
- Core SDK requires OkHttp 4.x → Framework bundles OkHttp 3.x

**Resolution Options:**

1. **Shade/relocate** the conflicting dependency in the wrapper
2. **Exclude** the core SDK's transitive and use the framework's version (if compatible)
3. **Upgrade** framework to a version that uses a compatible dependency version
4. **Reject** if conflict is irreconcilable

**Risk if ignored:** `NoSuchMethodError`, `ClassNotFoundException` at runtime.

---

### CRIT-004: Security Model Incompatibility

**Detect:** Core SDK's security requirements conflict with the target framework's capabilities.

**Examples:**

- Core SDK requires mTLS → Framework doesn't support custom SSL contexts
- Core SDK uses `char[]` for passwords → Framework passes `String` everywhere

**Resolution Options:**

1. **Adapter** that converts between security models
2. **Document limitation** with security implications
3. **Reject** if security downgrade is unacceptable

**Risk if ignored:** Security vulnerabilities, credential leaks.

---

## Major Anomalies

> **Warn user and get explicit acknowledgment.** These require design decisions.

### MAJ-001: Error Hierarchy Mismatch

**Detect:** Core SDK's exception types don't map to the target framework's error handling paradigm.

**Examples:**

- Core SDK uses checked exceptions → Framework uses unchecked/Result types
- Core SDK error hierarchy (10+ exception types) → Framework expects 3-4 HTTP error categories
- Core SDK has dual error hierarchies (checked + unchecked) — like auth0-java

**Resolution Options:**

1. **Flatten** core SDK exceptions into framework categories with original exception as cause
2. **Preserve** full hierarchy and let framework handler sort
3. **Translate** to framework-specific error types

---

### MAJ-002: Configuration Model Gap

**Detect:** Core SDK configuration approach is fundamentally different from the framework's.

**Examples:**

- Core SDK uses programmatic Builder → Framework uses YAML/HOCON config files
- Core SDK requires config at construction time → Framework uses lazy DI
- Core SDK has per-request config → Framework has global-only config

**Resolution Options:**

1. **Bridge** config file values to Builder parameters
2. **Expose** both: config file for common settings, programmatic for advanced
3. **Document** the translation clearly

---

### MAJ-003: Thread-Safety Model Mismatch

**Detect:** Core SDK client's thread-safety assumptions don't match the framework's concurrency model.

**Examples:**

- Core SDK client is NOT thread-safe → Framework shares singleton across threads
- Core SDK client IS thread-safe → Framework creates per-request instances (wasteful)
- Core SDK uses ThreadLocal → Framework uses coroutines/virtual threads

**Resolution Options:**

1. **Singleton with synchronization** if core SDK is not thread-safe
2. **Pool** of core SDK client instances
3. **Per-request** creation (if lightweight)

---

### MAJ-004: Lifecycle Mismatch

**Detect:** Core SDK client lifecycle doesn't align with the framework's lifecycle.

**Examples:**

- Core SDK client has no cleanup → Framework expects `Closeable`/`dispose()`
- Core SDK client caches state → Framework hot-reloads configuration
- Core SDK client has startup cost → Framework expects instant availability

**Resolution Options:**

1. **Wrap** with lifecycle hooks (startup/shutdown)
2. **Lazy init** on first use
3. **Health check** endpoint for readiness

---

### MAJ-005: Pagination Model Conflict

**Detect:** Core SDK's pagination approach doesn't fit the framework's conventions.

**Examples:**

- Core SDK uses cursor pagination → Framework expects offset/limit
- Core SDK returns `Iterator` → Framework expects `Stream` / reactive
- Core SDK requires manual page fetching → Framework auto-paginates

**Resolution Options:**

1. **Adapter** that translates pagination styles
2. **Expose both** cursor and offset APIs
3. **Auto-paginate** wrapper that fetches all pages

---

### MAJ-006: Serialization Incompatibility

**Detect:** Core SDK and framework use different serialization libraries or approaches.

**Examples:**

- Core SDK uses Jackson → Framework uses Gson
- Core SDK uses custom serializers → Framework expects standard types
- Core SDK's DTOs have Jackson annotations → Framework's JSON layer ignores them

**Resolution Options:**

1. **Use core SDK's DTOs directly** (if framework can handle them)
2. **Create adapter DTOs** with framework-appropriate annotations
3. **Register** core SDK's ObjectMapper with framework

---

## Minor Anomalies

> **Document in generated README.** These are noted but don't block generation.

### MIN-001: Naming Convention Difference

**Detect:** Core SDK uses different naming conventions than the target framework.

**Examples:**

- Core SDK uses `camelCase` → Framework convention is `snake_case`
- Core SDK uses `get/set` prefixes → Framework uses bare names
- Core SDK uses verbose names → Framework prefers terse names

**Resolution:** Follow framework conventions in wrapper API, map internally.

---

### MIN-002: Missing Feature Equivalent

**Detect:** Core SDK has a feature with no direct framework equivalent, but it's not essential.

**Examples:**

- Core SDK has admin-only features → Framework wrapper is for end-user apps
- Core SDK has telemetry → Framework doesn't have a telemetry hook

**Resolution:** Omit from wrapper with documentation explaining why.

---

### MIN-003: Performance Characteristics Difference

**Detect:** Core SDK's performance model differs from what framework users expect.

**Examples:**

- Core SDK makes synchronous HTTP calls → Framework users expect non-blocking
- Core SDK doesn't pool connections → Framework expects connection reuse
- Core SDK has high startup cost → Framework expects fast cold start

**Resolution:** Document in README, suggest configuration tuning.

---

### MIN-004: Logging Framework Difference

**Detect:** Core SDK uses a different logging framework than the target framework.

**Examples:**

- Core SDK uses SLF4J → Framework uses java.util.logging
- Core SDK uses console.log → Framework uses structured logging

**Resolution:** Bridge via logging facade or adapter.

---

### MIN-005: Date/Time Library Difference

**Detect:** Core SDK and framework use different date/time representations.

**Examples:**

- Core SDK uses `java.util.Date` → Framework uses `java.time.Instant`
- Core SDK uses epoch millis → Framework uses ISO-8601 strings

**Resolution:** Convert at the wrapper boundary.

---

## Anomaly Report Template

When reporting anomalies, use this format:

```markdown
### [SEVERITY-CODE] Title

**Detected:** [What was found]
**Core SDK:** [Relevant core SDK behavior]
**Target Framework:** [Relevant framework behavior]
**Impact:** [What happens if not resolved]
**Recommended Resolution:** [Best option]
**Alternative Resolutions:** [Other options]
**User Decision:** [Pending / Chosen option]
```
