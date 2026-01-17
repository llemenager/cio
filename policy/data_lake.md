# Data Lake Policy

## Status

**Adopted**

---

## Purpose

This policy defines the **role, ownership, and constraints** of the Data Lake within the system architecture.

The Data Lake exists to:

* Preserve **durable evidence** of external reality
* Enable **replay, recomputation, and auditability**
* Protect the correctness of downstream **OLTP decisions**

The Data Lake is **not** a system of record, a metrics store, or a business logic layer.

---

## Core Principle

> **The Data Lake stores evidence, not truth.**

Truth is decided downstream by OLTP systems.
The Data Lake exists so those decisions can be revisited safely.

---

## Ownership

### Product Engineering owns the Data Lake.

Ownership includes responsibility for:

* What data is ingested
* What data is retained
* Retention horizons
* Evidence schemas and contracts
* Replay guarantees
* Correctness risk arising from missing or incorrect evidence

### Data Engineering does **not** own retention decisions.

Data Engineering may:

* Design ingestion pipelines
* Optimize storage and compute
* Propose retention or compression strategies
* Surface cost and operational risks

But **may not unilaterally decide** what evidence is retained or discarded.

---

## Rationale (Non-Normative)

* Product Engineering is accountable for user-facing correctness.
* Only Product Engineering can judge whether evidence is required to:

  * Rebuild state
  * Explain behavior
  * Defend decisions
* Separating retention authority from correctness accountability creates organizational risk.

---

## Definition: Data Lake

A Data Lake is a storage layer that satisfies **all** of the following:

* Append-only
* Immutable (no updates or deletes)
* Payload-oriented (raw or lightly wrapped)
* Source-preserving
* Time-ordered
* Fully replayable

A Data Lake **may** be physically co-located with OLTP (e.g., in Postgres) but must remain **logically separate**.

---

## Allowed Contents

The Data Lake may contain:

* Raw external payloads (APIs, webhooks, file drops)
* Ingestion receipts
* Event logs
* Snapshots of upstream systems
* Backfill and historical imports
* Semi-structured data (e.g., JSON)

Each record **must** include:

* Source system identifier
* Ingestion timestamp
* Run / batch identifier
* Raw payload (unaltered)

---

## Explicit Non-Goals

The Data Lake must **not**:

* Act as a system of record
* Contain canonical business entities
* Encode business logic
* Support updates or corrections in place
* Be queried directly by product logic or UI
* Serve as a metrics or reporting source

---

## Authority Boundary

### Authority Hierarchy

```
Data Lake (Evidence, Non-Authoritative)
        ↓
OLTP (Authoritative Record)
        ↓
OLAP (Authoritative Served Metrics)
```

Authority **never flows upstream**.

---

## Access Rules

* The Data Lake is **read-only** outside ingestion pipelines.
* Only ingestion systems may write to the Data Lake.
* OLTP systems may read from the Data Lake for interpretation and replay.
* Downstream systems must not depend on the Data Lake for business answers.

---

## Immutability Contract

* Records in the Data Lake **must never be updated or deleted**.
* Corrections must be represented as **new records**, not mutations.
* If deletion is required (e.g., legal), it must:

  * Be explicit
  * Be logged
  * Trigger a documented correctness review

---

## Retention Policy

Retention is **intentional**, not default.

For each Data Lake source, Product Engineering must declare:

* Retention duration
* Justification for retention
* Replay dependency (what breaks if deleted)
* Earliest safe expiration point

“Keep everything just in case” is **not** a valid justification.

---

## Replay & Recomputability Requirements

All downstream systems that consume the Data Lake must support:

* Deterministic reprocessing from a given run/version
* Explicit logic versioning
* Clear lineage from lake records to derived state

If replay is not possible, the Data Lake is being misused.

---

## Naming & Schema Rules

To prevent authority creep:

* Data Lake schemas/tables must be named to reflect **evidence**, not entities

  * e.g., `*_payloads`, `*_events`, `*_receipts`
* Avoid names implying correctness or finality

  * ❌ `transactions_raw`
  * ✅ `<source>_transaction_payloads`

Schema documentation must clearly state:

> “This data is non-authoritative and may be incorrect.”

---

## Physical Co-location Rule

The Data Lake may share infrastructure with OLTP **only if**:

* Logical separation is enforced
* No foreign keys reference Data Lake tables
* No business logic reads Data Lake tables directly
* The Data Lake could be moved elsewhere without breaking correctness

If this is no longer true, separation is required.

---

## Exception Process

Exceptions to this policy require:

1. Written justification
2. Explicit benefit gained
3. Risk introduced
4. First failure mode at scale
5. Exit or migration plan

Exceptions must be approved by Product Engineering leadership.

---

## Summary (Executive)

* The Data Lake is an evidence archive owned by Product Engineering.
* It exists to protect correctness, enable replay, and preserve history.
* It is immutable, non-authoritative, and intentionally retained.
* Decisions live downstream; evidence lives here.