# Technical Approach & Architecture — NiyamSetu
### Regulation-Aware Workflow Orchestration for AICTE Approval Processing
**SIH26262 · AICTE, Ministry of Education**

---

## 1. System Architecture

```
                         ┌──────────────────────────────────┐
                         │        Presentation Layer          │
                         │  React + TypeScript + Tailwind     │
                         │  Role-scoped SPA: Institution,      │
                         │  Regional Officer (RO), EVC/SAC,    │
                         │  Regulation Bureau                  │
                         └───────────────┬────────────────────┘
                                         │ HTTPS / REST (OpenAPI)
                         ┌───────────────▼────────────────────┐
                         │         Application Layer           │
                         │  FastAPI (Python, async)            │
                         │  ├─ Case Registry & Routing Engine   │
                         │  ├─ Statutory Workflow State Machine │
                         │  ├─ Deadline & Escalation Scheduler  │
                         │  ├─ Obligation & Document Ledger      │
                         │  ├─ Bounded OCR / Evidence Extraction │
                         │  ├─ Case Rehearsal Sandbox Engine      │
                         │  └─ Signed Audit Ledger Service        │
                         └───────┬───────────────┬────────────────┘
                                 │               │
                 ┌───────────────▼───┐   ┌───────▼──────────────────┐
                 │  PostgreSQL          │   │  Rehearsal Schema           │
                 │  (operational system │   │  (isolated least-privilege  │
                 │  of record: cases,   │   │  DB role; simulation writes │
                 │  routes, ledger)     │   │  never reach production)    │
                 └───────────────────────┘   └──────────────────────────┘
                                 │
                         ┌───────▼────────────┐
                         │  Encrypted Document  │
                         │  Store (AES-256-GCM,  │
                         │  SHA-256 fingerprint) │
                         └───────────────────────┘
                                 │
                         ┌───────▼────────────────────┐
                         │  Mock AICTE / NIC-SMS /      │
                         │  WhatsApp / Identity Adapters │
                         │  (clearly badged, swappable)  │
                         └────────────────────────────────┘
```

**Design intent:** one authoritative backend, one relational store for operational data, and a database-enforced *second* schema purely for rehearsal simulation — so the system's flagship capability (test a hypothetical change) is provably incapable of mutating a real case, not just conventionally prevented by application code.

---

## 2. Technology Stack & Rationale

| Layer | Technology | Why this, specifically |
|---|---|---|
| Frontend | React 18 + TypeScript + Vite + Tailwind | Typed, role-scoped views for four distinct user types (Institution, RO, EVC/SAC, Bureau) |
| Backend | FastAPI (Python 3.11+) | Async I/O for concurrent document/notification processing; auto-generated OpenAPI spec is a live, inspectable technical artifact for judges |
| Database | PostgreSQL 15+ | ACID guarantees for statutory records; native recursive queries for routing/lineage traversal; **schema-level role isolation**, the mechanism the Rehearsal Sandbox depends on |
| ORM / Migrations | SQLAlchemy 2.x + Alembic | Versioned, reviewable schema evolution — no undocumented production schema drift |
| Document storage | Local encrypted volume, S3-compatible interface | AES-256-GCM at rest; swappable to managed object storage without an application rewrite |
| OCR / Extraction | Bounded, source-cited text/field extraction | Every extracted value keeps its source document, page and confidence — never silently promoted to "verified" |
| Scheduler | In-process durable worker, idempotency-keyed | Deduplicated maturity/deadline alerts without a Redis/Celery dependency |
| Auth | Argon2id password hashing, JWT access + rotating refresh tokens, mandatory TOTP 2FA for RO/EVC-SAC/Bureau roles | Strong credential hygiene with minimal operational surface |
| Audit integrity | Hash-chained ledger entries, each additionally Ed25519-signed | Tampering requires forging a private key, not just recomputing a hash — stronger than a hash-only chain |
| Rehearsal isolation | Dedicated PostgreSQL schema + least-privilege DB role, verified by an automated isolation check | The core engineering guarantee behind the system's flagship USP |

**Why not a larger stack (Celery, Redis, Keycloak, microservices, a 3D rendering layer):** every additional component is a point of demo failure and a line nobody on the team can defend under judge questioning. This stack is the smallest set of components that still satisfies every requirement of the problem statement end to end.

---

## 3. Data Model (Core Entities)

| Entity | Key Fields | Notes |
|---|---|---|
| **Institution** | id, name, region, authorized_contact_id | Tenant boundary for row-level isolation |
| **Application (Case)** | id, institution_id, category (LOA / EOA_STANDARD / EOA_3YEAR_MERIT / INFRA_MODIFICATION), workflow_version, current_stage, status | `category` selects the pinned, versioned route at creation time |
| **WorkflowDefinition** | id, category, version, stages, allowed_transitions, effective_from, approved_by | Immutable once published; a case keeps the version it was created under |
| **StageInstance** | id, application_id, stage_name, entered_at, deadline_at, owner_role | One row per stage visit; ageing is computed from `entered_at` |
| **TransitionRecord** | id, application_id, from_stage, to_stage, actor_id, reason, timestamp | Append-only; mandatory `reason` on any return/revert |
| **Obligation (Undertaking)** | id, application_id, condition_text, due_at, status, verification_actor | Submitted vs. verified kept as distinct states, never merged |
| **DeclaredField / ExtractedField** | field_name, declared_value, extracted_value, source, confidence | The dual-confidence model — every fact carries provenance |
| **Document** | id, application_id, type, issue_date, expiry_date, sha256_hash, review_status | Full renewal lineage via `supersedes_document_id` |
| **EVCEvent** | id, application_id, status, scheduled_at, geotag (simulated), report_status | A completed visit is never conflated with an accepted report |
| **NotificationEvent / NotificationDelivery** | id, channel, status (Sent/Pending/Failed/Simulated), dedup_key | Channel-level honesty labeling, deduplicated |
| **Escalation** | id, trigger_event_id, level, recipient_role, resolved_at | Configurable, versioned ladder |
| **AuditEntry** | id, prev_hash, entry_hash, signature, actor_id, action, timestamp | Immutable, insert-only at the database role level |
| **RehearsalSession** | id, application_id, baseline_snapshot_ref, hypothetical_changes, computed_diff, production_modified (always `false`) | Lives entirely in the isolated rehearsal schema |

---

## 4. Core Algorithms & Engines

### 4.1 Statutory Workflow State Machine
Each `WorkflowDefinition` is a directed graph of stages and guarded transitions, loaded by `category`. A transition fires only if:
1. The actor is authenticated and holds the required role.
2. The case is within the actor's regional/institutional scope.
3. The current stage matches the expected prior stage.
4. The requested transition is permitted by the case's **pinned** workflow version.
5. All transition prerequisites (documents, obligations, deficiency clearance) are satisfied.
6. A reason is present for any return/revert.

The transition, resulting assignment, and audit entry are committed in one transaction — no partial state is ever visible.

### 4.2 Case Rehearsal Sandbox (the flagship engine)
Unlike a rule-only "what-if" checker, the Sandbox simulates the **entire case state machine**:
```
1. Snapshot the case's current state (workflow stage, deadlines, obligations,
   documents, EVC eligibility) into the isolated rehearsal schema — read-only
   copy, never a live reference.
2. Apply the officer's hypothetical change (e.g., a corrected lease date)
   only inside that schema.
3. Re-evaluate: compliance rules, stage-transition eligibility, deadline
   recalculation, EVC eligibility, and which notifications would fire.
4. Return a structured diff: {rule_changes, stage_eligibility_changes,
   deadline_changes, notification_preview}.
5. Discard or persist the session for review — production rows are never
   touched. production_modified is hard-coded false and verified by an
   automated isolation test, not merely asserted.
```
This is the single largest technical differentiator versus the closest competitor's rule-only rehearsal engine.

### 4.3 Deadline & Escalation Scheduler
A durable, idempotency-keyed worker periodically scans for: approaching deadlines, newly overdue cases, unresolved escalations, and expiring documents. Each check writes a deduplicated event keyed on `(case_id, threshold)`, so a worker restart or a missed run cannot create a duplicate or lost alert — a catch-up scan recovers any gap.

### 4.4 Signed Audit Ledger
```
entry_hash_i = SHA256(prev_hash_(i-1) || canonical_json(event_i))
signature_i  = Ed25519_Sign(private_key_role, entry_hash_i)
```
Verification recomputes the chain from genesis and checks every signature against the signer's public key, reporting the exact offending entry if a single byte has changed anywhere in the history.

---

## 5. Constraints & Rules

- **Statutory routing cannot be invented.** Every route is loaded from an approved `WorkflowDefinition`; the prototype's four routes (LOA, EOA-Standard, EOA-3-Year-Merit, Infra Modification) are labeled prototype patterns pending authorized APH clause mapping.
- **No silent rule edits.** A `WorkflowDefinition` or deadline policy moves through `Draft → Review → Approved → Published → Superseded`; a developer cannot patch a live statutory deadline directly.
- **Rehearsal cannot leak into production.** Enforced at the database role level (separate schema, least-privilege grants), not only by application logic — provable by an automated isolation check, not just documentation.
- **Evidence promotion requires a human.** An OCR-extracted or declared value can never transition to "verified" without an explicit action by an authorized reviewer.
- **Tenant isolation.** Every query touching institution-scoped data is filtered server-side by the authenticated user's institution/region; cross-tenant access attempts are themselves logged as audit events.
- **Explicit provenance labeling.** Any UI element showing a channel or integration status (SMS, WhatsApp, bank confirmation, AICTE system sync) must render whether it is live or simulated — never ambiguous.
- **Immutability.** `AuditEntry` and `TransitionRecord` rows are insert-only; no UPDATE/DELETE grants exist on those tables at the database role level.

---

## 6. Non-Functional Requirements

| Requirement | Target | Approach |
|---|---|---|
| Availability (pilot scale) | Standard single-region deployment | Stateless API layer behind a reverse proxy; scheduled DB backups |
| Data integrity | Zero undetected tampering | Signed, hash-chained ledger, verified on demand |
| Confidentiality | Encrypted at rest and in transit | AES-256-GCM for documents, TLS for transport, hashed/salted credentials |
| Auditability | Every state change traceable to an actor, reason, and timestamp | Mandatory audit-entry write on every lifecycle transition |
| Rehearsal safety | Zero cross-schema writes, provably | Database-role-enforced isolation, automated verification script |
| Explainability | Every automated flag/escalation has a human-readable reason and rule reference | Deterministic, versioned rule evaluation — never an opaque black-box decision |

---

## 7. Assumptions & Dependencies

- Officials and institutions access the portal via standard browsers; no native mobile client is assumed necessary for the prototype.
- SMS/WhatsApp delivery is **not** assumed live — these channels run through a clearly labeled simulated outbox, swappable for an authorized NIC/Meta Business API integration later.
- No assumption is made of pre-existing API access to AICTE's internal systems; the integration adapter is a mock with synthetic records and real reconciliation logic, not a placeholder pretending to be connected.
- The exact current APH edition, clause numbers, and official routing matrix require confirmation by an authorized AICTE policy owner before any rule is treated as more than a prototype assumption.

---

*This document defines the technical approach only. For how this architecture solves the stated problem and what sets it apart, see `SIH26262-sol.md`. For feasibility, risk, and impact, see `SIH26262-Feasible.md`.*
