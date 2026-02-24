# SDK Wrapper Skill — Orchestrator

This document defines the routing logic for the SDK Wrapper Skill. The Copilot agent uses this to determine which stage to execute and how to transition between stages.

## Entry Points

### Full Workflow

**Trigger phrases:**

- "Use the SDK Wrapper Skill to generate a framework SDK"
- "Generate a wrapper SDK for {framework}"
- "Create a {framework} SDK on top of {core SDK}"

**Action:** Start at Stage 1.

### Resume Workflow

**Trigger phrases:**

- "Continue with Stage {N}"
- "I've approved the {artifact}, proceed"
- "Stage {N} looks good, next"

**Action:** Verify previous stage artifact exists and was approved, then execute Stage N.

### Single Stage

**Trigger phrases:**

- "Run Stage 1: Discover {repo URL}"
- "Just do the feasibility analysis"
- "Skip to code generation"

**Action:** Execute the specified stage. Warn if prerequisites are missing.

## State Machine

```
┌─────────────────────────────────────────────────────────────┐
│                                                              │
│  [START] ──→ Stage 1: Discovery                             │
│                  │                                           │
│              ✅ Approved                                     │
│                  │                                           │
│              Stage 2: Target Analysis                        │
│                  │                                           │
│              ✅ Approved                                     │
│                  │                                           │
│              Stage 3: Feasibility                            │
│                  │                                           │
│              ✅ Approved (all decisions made)                │
│                  │                                           │
│              Stage 4: Architecture                           │
│                  │                                           │
│              ✅ Approved                                     │
│                  │                                           │
│              Stage 5: Code Generation                        │
│                  │                                           │
│              ✅ Approved (compiles)                          │
│                  │                                           │
│              Stage 6: Test Generation                        │
│                  │                                           │
│              ✅ Approved (tests pass)                        │
│                  │                                           │
│              Stage 7: Update Strategy                        │
│                  │                                           │
│              ✅ Approved ──→ [COMPLETE]                      │
│                                                              │
│  At any stage:                                               │
│    🔄 "Revise" → Re-run current stage with feedback         │
│    ⏪ "Go back" → Re-run previous stage                     │
│    ❌ "Cancel" → End workflow                                │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

## Stage Prerequisites

| Stage | Requires                             | Produces                                |
| ----- | ------------------------------------ | --------------------------------------- |
| 1     | Core SDK repo URL                    | `docs/discovery-report.md`              |
| 2     | Approved discovery report            | `docs/target-analysis.md`               |
| 3     | Approved discovery + target analysis | `docs/feasibility-report.md`            |
| 4     | Approved feasibility (all decisions) | `docs/architecture.md`                  |
| 5     | Approved architecture                | Source code tree                        |
| 6     | Approved source code (compiles)      | Test suite                              |
| 7     | Approved tests (pass)                | `docs/update-strategy.md`, CI workflows |

## Approval Protocol

After each stage, the agent MUST:

1. **Present the artifact** — Show file path and key highlights
2. **Ask specific questions** — Not just "is this OK?" but targeted questions about accuracy
3. **Wait for explicit approval** — One of:
   - "Approved" / "Looks good" / "LGTM" / "Proceed" → Move to next stage
   - "Revise {specific thing}" → Re-run stage with feedback
   - "Go back to Stage {N}" → Revisit earlier stage
   - "I have a question about {X}" → Answer, then re-ask for approval

4. **Never auto-approve** — Even if the user says "do everything," stop after each stage

## Error Recovery

### Stage fails to produce output

- Log the error
- Explain what went wrong
- Ask user for additional context or alternative approach

### User provides conflicting input

- Surface the conflict explicitly
- Ask for clarification
- Don't guess

### Core SDK has unusual architecture

- Note it as an anomaly
- Ask user if this is expected
- Adapt the stage output accordingly

## Context Persistence

Between stages, maintain context by:

1. Referencing previous stage artifacts (read `docs/*.md`)
2. Carrying forward user decisions from Stage 3
3. Using approved architecture from Stage 4 as the blueprint for Stages 5-7

The agent should read the latest version of each artifact file before proceeding, in case the user edited it manually between stages.

## Multi-SDK Workflow

If the user wants to generate wrappers for multiple frameworks:

1. Complete the full 7-stage workflow for the first framework
2. For subsequent frameworks, **reuse Stage 1** (core SDK discovery doesn't change)
3. Start from Stage 2 with the new framework
4. Note shared patterns across generated wrappers in the update strategy

## Quality Gates

The agent should enforce these gates (warn, don't block):

| Gate                           | Stage | Check                                       |
| ------------------------------ | ----- | ------------------------------------------- |
| API Surface ≥ 5 public methods | 1     | Warn if SDK seems too small to wrap         |
| Coverage ≥ 60%                 | 3     | Warn if feasibility is low                  |
| No critical anomalies          | 3     | Block until resolved                        |
| Code compiles                  | 5     | Strongly recommend fixing before proceeding |
| Tests pass                     | 6     | Strongly recommend fixing before proceeding |
| All decisions recorded         | 3     | Block until all ❌ features have decisions  |
