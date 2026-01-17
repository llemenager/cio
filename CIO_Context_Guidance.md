# Project Context & Guidance Contract

## Purpose

This project exists to **calibrate engineering judgment**.

There is no single artifact, product, or end-state deliverable. Success is measured by how well our discussions, designs, and tradeoffs align with **real-world best practices at scale**, and how effectively we reason our way *down* from those practices when constraints make them inappropriate.

This is a thinking system, not a shipping system.

---

## Thesis

We start from **what is correct at absolute scale**—large datasets, long histories, distributed teams, strict correctness requirements—and then deliberately trade down complexity **only when it stops making sense**.

Every simplification must be:

* Explicit
* Justified
* Reversible where possible

---

## Non-Goals

This project is **not**:

* A polished SaaS or consumer application
* A performance benchmark or scale demo
* A place to prematurely optimize UX
* A showcase of novelty for its own sake

If something exists here, it exists to sharpen judgment, not to impress.

---

## Product-Agnostic Default
All guidance in this repository is intentionally product-agnostic. Principles, policies, and examples are framed to reflect general best practices that hold across domains, organizations, and technology stacks. No document here should be interpreted as prescribing a specific product, internal system, or proprietary architecture. When examples are used, they are illustrative only and must not be treated as normative implementations. The default assumption is that these practices should be defensible in any senior-level design review, independent of the particular product or codebase in which they are applied.

---

## Operating Principles

* **Correctness > cleverness**
* **Auditability beats convenience**
* **Explicit tradeoffs over implicit assumptions**
* **Simple and reversible beats elegant and fragile**
* **Local-first by default; distribution must be justified**
* **If a decision would be hard to defend in a senior design review, it’s probably wrong**

---

## Best-Practice Reference Frame

Guidance should assume:

* Practices that hold up at **large scale** (data volume, time horizon, team size)
* Patterns defensible in **senior/principal-level design reviews**
* Awareness of how and *why* those patterns break down at smaller scales

We do **not** start from “what’s easiest locally” and work upward.
We start from “what survives reality” and work downward.

---

## Constraints & Realities

* Solo development, limited time
* Long-lived, messy, human-generated data
* Heavy use of semi-structured data (JSON)
* Historical backfills and recomputation are inevitable
* Decisions are **sticky** by intent — we aim to “do it right the first time”
* We explicitly avoid getting “into the mud” with unplanned complexity

---

## Quality Bar for Guidance

Good guidance must include:

1. **Multiple viable approaches**
2. **Tradeoffs and failure modes**
3. **Clear recommendation**
4. **Conditions under which the recommendation changes**
5. **Migration or rollback considerations**

Advice that simply says “use X” without explaining *why*, *when*, and *when not* is insufficient.

---

## AI Collaboration Contract

AI is an **active collaborator**, not a passive assistant.

AI **may**:

* Propose architectural changes
* Rewrite core logic
* Flag smells even without a ready solution
* Decide when explicitly asked to decide

AI should:

* Default to **conservative, production-grade advice**
* Offer experimental or novel ideas **clearly labeled as such**
* Challenge over-engineering drift
* Introduce novelty when it meaningfully expands the design space

AI should **avoid**:

* Cargo-cult patterns
* Premature service sprawl
* Suggesting specific cloud providers unless the discussion explicitly requires it

---

## Architectural Snapshot (High Level)

* Canonical data flows favor determinism, idempotence, and auditability
* Raw → staging → canonical/fact → derived/reporting layers
* Provenance is preserved even when data is squashed or canonicalized
* Recomputability is a first-class concern
* “Truth” is explicit; derivations are treated as such

Details evolve, principles do not.

---

## Canonical Vocabulary

This project uses a shared vocabulary to reduce ambiguity.
Key terms (non-exhaustive):

* **Canonicalization** – Making equivalent data representations deterministic
* **Anchor** – The authoritative record chosen among equivalents
* **Squash** – Collapsing multiple representations into one canonical entity
* **Run ID** – A unit of execution with replay semantics
* **Stg / Fact / Dim / Rpt** – Semantic layers, not just schemas

(See `glossary.md` for definitions as the vocabulary grows.)

---

## Decision Records

Architectural decisions are captured as short, explicit records under `decisions/`.

Each record should answer:

* What decision was made?
* What alternatives were considered?
* Why this choice?
* What would cause us to revisit it?

Decisions are expected to be **defensible**, not immutable.

---

## Calibration Anchor Example

A recurring tension in this project:

> **Graph database exploration vs relational purity**

* Graph models may better express relationships, provenance, and inference.
* Relational models excel at determinism, auditability, and operational simplicity.

Discussions should:

* Start from how this choice looks at large scale
* Explicitly justify any deviation
* Be honest about operational and cognitive cost
* Avoid novelty unless it earns its keep

This tension is intentional and serves as a standing calibration exercise.

---

## How to Engage with This Repo

When contributing guidance or changes:

* Assume the reader is a senior engineer
* Make assumptions explicit
* State when advice is conservative vs exploratory
* Prefer clarity over exhaustiveness
* If something feels “too clever,” call it out

---

### Closing Note

This repository is a **living calibration tool**.

If discussions here consistently mirror how good systems are designed, debated, and evolved in the real world, the project is succeeding—regardless of what it produces.