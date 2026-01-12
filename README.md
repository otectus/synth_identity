# SynthIdentity

![Version](https://img.shields.io/badge/version-0.1.0-blue)
![Python](https://img.shields.io/badge/python-3.10%2B-brightgreen)
![License](https://img.shields.io/badge/license-MIT-lightgrey)

**SynthIdentity** is the identity governance subsystem of the Nexus architecture.
It defines, enforces, versions, and safely degrades an AI agent’s identity over time.

Unlike prompt-only “persona” systems, SynthIdentity treats identity as a **versioned, invariant-checked, auditable state machine** with explicit failure semantics.

---

## Core Responsibilities

SynthIdentity is responsible for:

* Defining an immutable **Identity Kernel** (core traits and values)
* Enforcing **invariants** that prevent identity drift or unsafe behavior
* Maintaining a **monotonic identity timeline**
* Supporting **human-in-the-loop approval workflows**
* Providing **graceful degradation** via a minimal fallback identity
* Producing **high-fidelity diagnostics** for audits and debugging

It does **not** generate text, store memories, or manage mood.
It exists to answer one question with certainty:

> “Who is this agent allowed to be *right now*?”

---

## Design Philosophy

SynthIdentity is built on four non-negotiable principles:

1. **Identity must be immutable**

   * Kernels cannot be edited in place
   * Change happens only via new snapshots

2. **Identity must be governable**

   * Approval states are explicit and typed
   * Rollbacks are first-class events

3. **Identity must be auditable**

   * Violations are structured, rule-addressable, and loggable
   * Version history is monotonic and tamper-resistant

4. **Identity must fail safely**

   * Any failure collapses to a known, minimal identity
   * The system always knows when it is degraded

---

## High-Level Architecture

```
IdentityKernel (immutable)
        ↓
InvariantEngine (validation)
        ↓
IdentitySnapshot (versioned state)
        ↓
IdentityManager (timeline + governance)
        ↓
SynthCore (orchestrator, external)
```

SynthIdentity is **purely deterministic**.
No learning, no probabilistic updates, no hidden mutation.

---

## Key Concepts

### IdentityKernel

The `IdentityKernel` represents the immutable “soul” of the agent.

It defines:

* Name and role
* Core values
* Communication style
* Areas of expertise
* A list of invariants (rules that must never be violated)

Once created, a kernel cannot be modified.

This guarantees that identity evolution is **additive and traceable**, not destructive.

Implementation reference: 

---

### Invariants

Invariants are rules that enforce identity boundaries.

Supported forms:

* **Callable predicates**: `(text: str) -> bool`
* **Declarative rules**: dictionaries with explicit rule IDs

Example rule types:

* `contains_not` – forbid specific patterns
* `contains` – require specific patterns

Every invariant violation:

* Is tagged with a **Rule ID**
* Produces a **structured diagnostic string**
* Is logged with full exception context if execution fails

If an invariant crashes, it is treated as a **hard violation**.

Invariant failures never fail open.

Implementation reference: 

---

### ApprovalStatus

Every identity snapshot carries an explicit approval state:

* `AUTO` – system-generated, unreviewed
* `REVIEWED` – reviewed by a system or moderator
* `USER_APPROVED` – explicitly approved by a human
* `SYSTEM_ROLLBACK` – emergency fallback state

Approval is not metadata.
It is a **governance signal** meant for downstream policy enforcement.

Implementation reference: 

---

### IdentitySnapshot

An `IdentitySnapshot` is a versioned, timestamped record of identity at a point in time.

Each snapshot contains:

* The immutable kernel
* A strictly monotonic version number
* A UTC timestamp
* An approval status
* A human-readable reflection describing *why* the snapshot exists

Snapshots are append-only.
History is never rewritten.

Implementation reference: 

---

### IdentityManager

The `IdentityManager` governs the identity timeline.

Responsibilities:

* Assigns versions internally (callers cannot set versions)
* Ensures versions always increase
* Stores per-user identity histories
* Enforces snapshot rotation limits
* Provides safe access to the latest identity

This prevents:

* Version drift
* Timeline forks
* Accidental regressions

Implementation reference: 

---

### Minimal Skeleton Identity (Fail-Safe)

If identity loading or validation fails, SynthIdentity collapses to a **minimal skeleton identity**.

This identity:

* Has a safe, generic kernel
* Is explicitly marked with `SYSTEM_ROLLBACK`
* Preserves system operability
* Signals downstream systems to degrade behavior

The system never runs without an identity.

Implementation reference: 

---

## Typical Lifecycle

1. Define an `IdentityKernel`
2. Validate generated text against invariants
3. Commit a new snapshot via `IdentityManager`
4. Snapshot receives an internally assigned version
5. Snapshot is optionally reviewed or approved
6. Downstream systems consume the latest approved identity
7. On failure, system falls back to the minimal skeleton

---

## Example Usage

```python
from synthidentity.kernel import IdentityKernel
from synthidentity.snapshot import IdentityManager, ApprovalStatus

kernel = IdentityKernel(
    name="Nova",
    role="synthetic assistant",
    core_values=["honesty", "clarity", "safety"],
    communication_style="direct",
    expertise_domains=["systems design", "reasoning"],
    invariants=[
        {"id": "no_illegal", "type": "contains_not", "pattern": "illegal"}
    ]
)

manager = IdentityManager()

snapshot = manager.commit_new_snapshot(
    user_id="user_123",
    kernel=kernel,
    reflection="Initial stabilized identity",
    status=ApprovalStatus.AUTO
)
```

---

## What SynthIdentity Does *Not* Do

SynthIdentity intentionally avoids:

* Learning or adapting identity automatically
* Editing identity in place
* Generating text
* Scoring emotions or mood
* Managing memory or context

Those concerns belong to other subsystems.

---

## Integration Notes

SynthIdentity is designed to be:

* Orchestrator-driven (via SynthCore)
* Storage-agnostic (in-memory now, database later)
* Deterministic and testable
* Safe to run in degraded environments

It pairs naturally with:

* SynthCore (orchestration)
* SynthMemory (long-term recall)
* SynthMood (affective modulation)

---

## Roadmap Alignment

Phase 1:

* Immutable kernels
* Invariant enforcement
* Snapshot versioning
* Rollback safety

Phase 2 (planned):

* Policy-driven approvals
* Identity diff tooling
* Cross-snapshot drift detection
* External audit pipelines

---

## License

MIT License.
See `LICENSE` for details.

---

## Final Note

SynthIdentity is not a “persona system.”
It is an **identity control plane**.

If you treat identity as a string, this is overkill.
If you treat identity as something that must *survive time, failure, and human scrutiny*, this is the minimum viable rigor.
