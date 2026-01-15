# Data Store Usage Policy

**Transactional vs Analytical vs Graph Databases**

## Status

**Adopted**

## Purpose

This policy defines **when and how different database paradigms may be used**, with the explicit goal of preserving:

* Correctness over time
* Auditability and explainability
* Recomputability
* Conceptual integrity as the system evolves

This policy assumes **large scale, long-lived data, and changing requirements**, even when current usage is small.

The intent is not to prevent experimentation, but to **prevent accidental authority**, architectural drift, and irreversibility.

---

## Core Distinction: Two Kinds of Authority

A key principle of this policy is that **“authoritative” is not a single concept**.

### 1. Authoritative Record (System of Record — SoR)

The system that owns:

* Canonical facts
* State transitions
* Historical truth

This is where you go to answer:

> “What actually happened, when, and why?”

**Properties**

* Auditable
* Correctable
* Replayable
* Deterministic

### 2. Authoritative Served Truth (System of Truth — SoT)

The system that owns:

* Official answers to *defined questions*
* Published metrics and contracts
* Vended datasets consumed by users or downstream systems

This is where you go to answer:

> “What is the official answer for this metric or question?”

**Properties**

* Versioned
* Documented
* Reproducible from SoR inputs
* Contractually stable

**Rule**

* A system may be authoritative-served **only if** its outputs are derivable from an authoritative-record system.

---

## Canonical Database Roles

Each database must have **exactly one primary role**.
Serving multiple primary roles requires explicit exception documentation.

---

### 1. Transactional Databases (OLTP)

**Primary Role:**

> System of Record (authoritative-record)

**Responsibilities**

* Canonical entities
* State transitions
* Ingestion and mutation logic
* Referential integrity
* Business events and lifecycle changes

**Allowed Business Logic**

* Operational logic that **changes state or creates obligation**, including:

  * Classification acceptance
  * Approval / locking
  * Ownership changes
  * Balance-affecting mutations

**Required Properties**

* Strong consistency
* Deterministic writes
* Explicit schemas
* Temporal correctness
* Explainable mutations

**Non-Goals**

* Large-scale aggregation
* Exploratory analytics
* Relationship inference
* Heuristic reasoning

**Policy**

* OLTP is the **only authoritative-record system**
* All other stores must be derivable from OLTP
* If OLTP data is deleted, the system is fundamentally broken

---

### 2. Analytical Databases (OLAP)

**Primary Role:**

> System of Truth for defined metrics (authoritative-served)

**Responsibilities**

* Aggregations
* Time-series analysis
* Reporting
* Trend analysis
* Metric publication

**Allowed Business Logic**
**Definitional metric logic**, including:

* Inclusion/exclusion rules
* Rollups and mappings
* Accrual and time-bucketing logic
* Metric semantics (“what does this number mean?”)

This logic **is business-critical** and **may be authoritative-served**.

**Required Properties**

* Deterministic query semantics
* Explicit dimensional modeling
* Versioned transformation logic
* Lineage back to SoR inputs
* Rebuildability from source data

**Non-Goals**

* Acting as a system of record
* Enforcing state transitions
* Accepting direct user writes
* Silent mutation of canonical facts

**Policy**

* OLAP outputs may be **the official answers vended to consumers**
* Authority is **delegated**, not original
* OLAP must fail “loudly” if inputs or definitions change

---

### 3. Graph Databases

**Primary Role:**

> Exploratory, inferential, or relational reasoning

**Responsibilities**

* Relationship traversal
* Identity resolution
* Dependency analysis
* Inference support
* AI / ML feature scaffolding

**Allowed Logic**

* Probabilistic relationships
* Heuristic groupings
* Traversal-based reasoning
* Non-deterministic inference

**Required Properties**

* Fast adjacency traversal
* Flexible relationship modeling
* Tolerance for partial or uncertain truth

**Explicit Limitations**

* Graph outputs are **advisory by default**
* Graphs do **not** own historical truth
* Graphs do **not** own metric authority

**Policy**

* Graph databases are **never authoritative-record**
* Graph databases are **not authoritative-served by default**
* They must be fully rebuildable from SoR / SoT inputs
* Deleting a graph must degrade *capability*, not *correctness*

Promotion of graph-derived conclusions into OLTP requires an **explicit, auditable workflow**.

---

## Business Logic Classification (Critical)

All logic must fall into **exactly one bucket**.

### Bucket A — Operational Business Logic (OLTP-only)

Logic that:

* Changes system state
* Creates or removes obligation
* Has legal, financial, or correctness implications

**Examples**

* Accepting a transaction classification
* Locking a record
* Posting a balance change

**Policy**

* Must live in OLTP
* Must be auditable
* Must be replayable

---

### Bucket B — Definitional Metric Logic (OLAP-allowed)

Logic that:

* Defines *what a metric means*
* Shapes how data is aggregated or interpreted
* Is consumed for decisions, reporting, or automation

**Examples**

* “Net spend excludes transfers”
* “Budget adherence uses accrual logic”
* “Active users definition v3”

**Policy**

* Expected to live in OLAP
* May be authoritative-served
* Must be versioned and documented

---

### Bucket C — Inferential / Probabilistic Logic (Graph / ML)

Logic that:

* Is heuristic or best-effort
* Produces likelihoods, not facts
* Improves insight but is not guaranteed

**Examples**

* Merchant deduplication suggestions
* Duplicate detection
* Category inference

**Policy**

* Advisory by default
* Cannot silently affect authoritative systems
* Promotion requires explicit acceptance + traceability

---

## Authority & Data Flow Rules

### Authority Hierarchy

```
OLTP (Authoritative Record)
        ↓
OLAP (Authoritative Served Metrics)
        ↓
Graph (Exploratory / Inferential)
```

Reversing this flow requires a documented exception.

---

### Write Rules

| Store Type | Direct business writes |
| ---------- | ---------------------- |
| OLTP       | Yes                    |
| OLAP       | No (ETL / ELT only)    |
| Graph      | No (projection only)   |

---

## Recomputability Contract

All non-OLTP systems must support:

* Cold rebuild from SoR inputs
* Versioned derivation logic
* Clear lineage
* Deterministic replays for a given version

If a system cannot be rebuilt, it is violating policy.

---

## Explicit Anti-Patterns

The following require escalation and written justification:

* Treating OLAP as a system of record
* Encoding state transitions in OLAP
* Encoding business rules inside graph traversals
* Allowing graph inference to silently affect balances, eligibility, or correctness
* Multiple implicit systems of record

---

## Allowed Exceptions

Exceptions are permitted **only if**:

1. Authority boundaries remain intact
2. The system degrades safely if deleted
3. The exception is documented with:

   * Benefit gained
   * Risk introduced
   * First failure mode at scale

---

## Summary (Executive Version)

> OLTP systems are the sole authoritative record of facts and state transitions. OLAP systems may be authoritative-served for published metrics, provided their logic is versioned and reproducible from canonical inputs. Graph databases are strictly non-authoritative and exist to support inference and exploration only. Authority is delegated downstream, never invented.