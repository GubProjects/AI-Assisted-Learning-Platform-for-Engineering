# Architecture

**AI-Assisted Learning Platform — Public System Overview**

This document describes the high-level architecture of the AI-Assisted Learning Platform. It is intended to explain system boundaries, control flows, and design principles without exposing internal implementation details.

> The platform is built around one idea: **Learning must be validated through execution, governed by deterministic controls, and supported by constrained AI.**

## 1. System Layers

The platform is organized into three logical planes:

```
┌─────────────────────────────┐
│       Client / UI           │
│  (Learner & Instructor UX)  │
└──────────────┬──────────────┘
               │
┌──────────────▼──────────────┐
│  Control & Orchestration    │
│   (Deterministic Core)      │
└──────────────┬──────────────┘
               │
┌──────────────▼──────────────┐
│   Execution & Evidence      │
│ (Sandboxes, Logs, State)    │
└─────────────────────────────┘
```

## 2. Major Subsystems

### 2.1 Client Layer

- Presents learning plans and steps
- Submits answers and execution requests
- Displays feedback and adaptive hints

The client is stateless with respect to learning logic. **All authority lives in the control plane.**

### 2.2 Control & Orchestration Plane

The control plane is the brain of the platform.

**Responsibilities:**
- Generate learning plans
- Enforce step order
- Evaluate results
- Control AI behavior
- Apply security & quota rules

**Key components:**

| Component | Responsibility |
|-----------|-----------------|
| Plan Orchestrator | Builds learning paths from templates |
| Step Engine | Tracks progression and gating |
| Evaluation Engine | Validates outputs deterministially |
| Policy Layer | Enforces security, quotas, boundaries |
| AI Mediation Layer | Sanitizes and constrains AI outputs |

**This layer contains no user-executable code.**

### 2.3 Execution & Evidence Plane

This plane performs untrusted work and produces proof:

| Component | Purpose |
|-----------|---------|
| Sandboxes | Execute learner code safely |
| Evidence Store | Logs, evaluation records |
| Dataset Store | Curated RAG sources |
| Audit Logs | Immutable activity trail |

All outputs flow back into the control plane for validation.

## 3. Learning Flow

```
Learner → Plan Request
        ↓
 Orchestrator builds plan
        ↓
  Step → Answer → Execute
        ↓
Deterministic Evaluation
        ↓
  Progress / Feedback
        ↓
 Adaptive Hints (RAG)
```

**The learner cannot advance unless deterministic gates pass.**

## 4. AI Integration Model

AI is not trusted.

It is:
- read-only to curated datasets
- mediated through policy
- never allowed to advance state

AI only influences presentation and hints, **never system truth.**

## 5. Security Boundaries

| Boundary | Protection |
|----------|-----------|
| UI → Control Plane | Auth, session validation |
| Control → Sandbox | Execution profiles |
| Sandbox → Platform | Output validation |
| AI → System | Sanitization + policy |

**Every transition is logged and validated.**

## 6. Sandbox Isolation

Sandboxes are:
- Ephemeral
- Resource-limited
- Profile-based
- Non-networked by default

Future designs include hardware isolation (gVisor / Firecracker).

## 7. Quota & Cost Controls

Enforced at the control plane:
- Per-user execution limits
- AI call quotas
- Sandbox runtime budgets

This prevents abuse and runaway costs.

## 8. Extensibility Model

Subjects are:
- Declarative data
- Validation rules
- Execution profiles

**No engine rewrite is required to add new domains.**

## 9. Control vs Workload Separation

| Plane | Role |
|-------|------|
| Control | Governs, validates, enforces |
| Workload | Executes, produces evidence |
| AI | Assists, never decides |

This prevents self-authorization and silent failure.

## 10. Design Goals

- **Determinism over magic** — Predictable behavior, no hidden logic
- **Controls over prompts** — Policy gates, not LLM confidence
- **Evidence over claims** — Proof of execution, not assertions
- **Safety over speed** — Correctness prioritized over latency

This architecture treats learning as a governed system, not a content feed.

---

**For implementation details**, refer to internal documentation.  
**For design questions**, open an issue or discussion.
