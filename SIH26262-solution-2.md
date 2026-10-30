# NiyamSetu
### Regulation-Aware Workflow Orchestration for AICTE Approval Processing
**SIH26262 · AICTE, Ministry of Education**

> *निyam + सेतु — the bridge between a regulatory rule and the action it governs. Every case on the right path. Every obligation on time. Every decision provably traceable.*

---

## 0. Why This Document Exists

Our team already has a strong 26-section NiyamSetu design. This revision sharpens it after benchmarking it against **six** competing public approaches to the same problem statement. One of them (the "Approval Crash-Test" team) has already built a rehearsal/sandbox engine with real database-level isolation — the exact category of idea NiyamSetu v1 held back as an optional Phase 6. That changes the competitive picture, and this document responds to it directly instead of pretending it doesn't matter.

---

## 1. Competitive Landscape

| Team | Core Strength | Where It's Exposed |
|---|---|---|
| **AICTE Smart Approval & Compliance Management System** | Polished 8-stage statutory pipeline, Recharts analytics, multi-channel alert center (In-App/Email/SMS/WhatsApp) with a "trigger test alert" demo button | Frontend-only state (local storage persistence) — no server-enforced authorization shown; "production backend" is a target architecture, not built |
| **AICTE Regulatory Control Tower** | Best statutory-routing specificity of any team — explicit per-category routes (New Institute / EOA-Standard / EOA-3-Year-Merit / Infrastructure Modification) mapped to real APH chapters; 8-region RBAC; geotagged EVC lock; fast-forward cron simulation | SHA-256 hash-only audit (no asymmetric signing); configurable rules API (`PUT /api/rules`) edits live SLA days with no visible approval/versioning gate before publishing |
| **AICTE Approval & Compliance Platform** | The most disciplined honesty practice of all six — every demo value explicitly labeled `DEMO_ASSUMPTION`; real OCR (pypdf + Tesseract) with mandatory human review; genuine source-of-truth/reconciliation workspace for a mock import; milestone-gated (M1–M6) build log | Heaviest setup/documentation surface for a 5-minute judge walkthrough; OCR/UI integration explicitly still pending at time of writing |
| **"Approval Crash-Test" (sih26262 monorepo)** | **Strongest technical build of the six.** A real Rehearsal Engine with PostgreSQL-enforced isolation across four least-privilege roles, immutable versioned rule grammar with three-valued logic, hash-chained tamper-evident audit, 838 passing tests, production-shaped Docker deployment with fail-fast config guards | Rehearsal is scoped to **compliance-rule outcomes only** — it does not simulate downstream effects on deadlines, EVC eligibility, or notifications; UI explicitly avoids calling it "replay," signaling the team itself knows its scope is narrower than full case simulation |
| **SIH26263 monorepo (3D/Three.js variant)** | Visually distinctive (WebGL shader backdrop, status gate rings); simplest-to-read route-code model (S/SE/ES/E/SES) with a clean 3-level escalation ladder (L1/L2/L3) | Shallow data model (integer status enums, string-only audit log, no RBAC enforcement visible beyond query filters) — strong visual polish, thin regulatory substance |
| **ApproveDesk** | Clean, minimal, role-scoped Next.js + FastAPI build; AI assistant explicitly scoped to "summarize grounded facts, never decide"; honest "Known Limitations" section | Narrowest feature surface of the six — no EVC module, no document-validity/renewal lifecycle, no rehearsal/sandbox concept at all |

### What this tells us
No competitor combines **statutory routing precision** (Regulatory Control Tower's strength), **full-case rehearsal rather than rule-only rehearsal** (going beyond Approval Crash-Test's scope), **discipline about what's real vs. assumed** (Approval & Compliance Platform's strength), and **a build scope that is still finishable and demonstrable in the available time.** That combination — not a longer feature list — is what NiyamSetu v2 is built around.

---

## 2. Design Principles (Revised)

1. **Statutory routing is data, not code.** Every application category's path is a versioned configuration object, never a hard-coded `if`.
2. **Rehearsal is a core differentiator, scoped wider than any competitor's.** Not just "will this rule pass" — "what happens to the whole case" if evidence or a rule changes.
3. **Separate the fact from the confirmation.** An uploaded value, an OCR-extracted value, and a human-verified value are three different fields, never collapsed into one.
4. **No silent authority.** Automation routes, reminds, and explains. It never grants, rejects, or finds non-compliance on its own.
5. **Every claim is labeled real or assumed**, in the interface itself — not just in a README a judge may not read.
6. **Terminology matches the Approval Process Handbook exactly**, so scoring against the official rubric is unambiguous.

---

## 3. Terminology (used precisely and consistently)

| Term | Meaning |
|---|---|
| **APH** | Approval Process Handbook — AICTE's governing procedural document |
| **LOA** | Letter of Approval — granted to a new technical institution |
| **EOA** | Extension of Approval — annual/multi-year continuation approval for an existing institution |
| **EOA-3-Year Merit Track** | Accelerated EOA route for institutions meeting NIRF/NBA/NAAC eligibility thresholds |
| **EVC** | Expert Visit Committee — conducts physical/virtual institutional inspection |
| **SAC / SHC** | Scrutiny Appellate Committee / Standing Hearing Committee — quasi-judicial hearing stage for flagged or deficient cases |
| **RO** | Regional Office — one of AICTE's regional administrative units |
| **Regulation Bureau** | AICTE headquarters function with national oversight authority |
| **Competent Authority** | The authorized decision-maker for a final statutory outcome |
| **TAT / SLA** | Turnaround Time / Service Level Agreement — the prescribed processing duration for a stage |
| **Undertaking** | A time-bound institutional obligation (e.g., an affidavit or compliance commitment) tied to an approval |

---

## 4. System Overview

NiyamSetu is a **workflow-orchestration and compliance-intelligence layer** sitting above AICTE's approval process. Its core differentiating engine is the **Case Rehearsal Sandbox** — not a rule-only "what-if," but a fully isolated, read-only snapshot of an entire case (workflow state, deadlines, EVC eligibility, obligations, and the notifications that would fire) that an officer can test hypothetical changes against before taking a real, audited action.

### 4.1 Why this beats the closest competitor's version
"Approval Crash-Test" proves hypothetical changes only against **compliance rule outcomes**. NiyamSetu's sandbox proves them against the **entire case state machine** — so an officer can ask "if this lease date is corrected, does the case also become EVC-eligible, does its SLA deadline shift, and what notification would this trigger?" in one rehearsal, not three separate manual checks. The isolation guarantee is the same database-enforced discipline (least-privilege roles, a separate schema for simulation writes) — applied to a strictly larger surface.

---

## 5. Architecture

```
                      ┌───────────────────────────────┐
                      │        Presentation Layer       │
                      │ React + TypeScript + Tailwind   │
                      │ Institution · RO · EVC/SAC ·    │
                      │ Regulation Bureau role views     │
                      └────────────────┬─────────────────┘
                                       │ HTTPS / REST (OpenAPI)
                      ┌────────────────▼─────────────────┐
                      │         Application Layer          │
                      │ FastAPI (async)                    │
                      │ ├─ Case Registry & Routing Engine   │
                      │ ├─ Statutory Workflow State Machine │
                      │ ├─ Deadline & Escalation Scheduler  │
                      │ ├─ Obligation & Document Ledger     │
                      │ ├─ Evidence Extraction (OCR, bounded)│
                      │ ├─ Case Rehearsal Sandbox Engine     │
                      │ └─ Signed Audit Ledger Service       │
                      └───────┬───────────────┬───────────────┘
                              │               │
                ┌──────────────▼──┐   ┌───────▼───────────────┐
                │  PostgreSQL       │   │ Rehearsal Schema        │
                │  (operational      │   │ (isolated, least-       │
                │  system of record) │   │ privilege DB role,      │
                │                    │   │ never touches production)│
                └────────────────────┘   └──────────────────────────┘
                              │
                      ┌───────▼────────────┐
                      │ Private Document    │
                      │ Store (encrypted,   │
                      │ SHA-256 fingerprint)│
                      └──────────────────────┘
                              │
                      ┌───────▼────────────────────┐
                      │ Mock AICTE/Bank/Identity     │
                      │ Adapters — clearly badged,    │
                      │ swappable for authorized live │
                      │ integration later              │
                      └────────────────────────────────┘
```

### 5.1 Stack Rationale

| Layer | Technology | Why |
|---|---|---|
| Frontend | React + TypeScript + Tailwind | Fast, typed, role-scoped views |
| Backend | FastAPI (async) | Live OpenAPI docs double as a judge-facing technical artifact |
| Database | PostgreSQL | ACID integrity; native recursive queries for routing/lineage; role-based schema isolation for the rehearsal sandbox |
| Auth | Argon2id + JWT + mandatory TOTP 2FA for RO/Bureau/Competent Authority roles | Credential hygiene without an external IdP dependency for the prototype |
| Scheduler | In-process durable worker with idempotency keys | Deduplicated reminders and escalations without a Redis/Celery dependency |
| Audit integrity | Hash-chained ledger, each entry Ed25519-signed | Stronger than a hash-only chain — tampering requires forging a private key, not just recomputing a hash |
| Rehearsal isolation | Dedicated PostgreSQL schema + least-privilege role, verified by an automated isolation check | Same rigor as the strongest competitor's approach, applied to full case simulation rather than rule output alone |

**Deliberately excluded for the prototype:** Celery/Redis, Keycloak, Three.js visual flourishes, microservices. Every added dependency is a point of demo failure; this stack is the smallest set of components that still satisfies every PS requirement and can be defended line-by-line under questioning.

---

## 6. Statutory Routing Matrix (Versioned Configuration, Not Hard-Coded Logic)

Each application category loads its own approved, versioned route:

```
NEW_INSTITUTE (LOA)
  SUBMITTED → SCRUTINY → [RE-SCRUTINY] → EVC → [SAC/SHC] → LOA DECISION

EOA_STANDARD
  SUBMITTED → SCRUTINY → EOA DECISION
  (routes to EVC only if a deficiency or undertaking breach is flagged)

EOA_3YEAR_MERIT
  SUBMITTED → AUTOMATED ELIGIBILITY CHECK (NIRF/NBA/NAAC thresholds, human-confirmed) → EOA DECISION

INFRASTRUCTURE_MODIFICATION
  SUBMITTED → EVC → [SAC/SHC] → DECISION
```

These are **prototype route patterns**, explicitly labeled as such in-app, pending confirmation of the exact current APH edition and clause mapping by an authorized policy owner — never presented as official AICTE routing.

Routes are published, immutable configuration objects (`workflow_id`, `version`, `effective_from`, `approved_by`). A case keeps the version it was created under unless explicitly, authorizedly migrated. No developer can silently edit a live statutory deadline.

---

## 7. Core Capabilities

1. **Case Registry** — one record per application, tenant-isolated by institution, searchable by region/stage/category.
2. **Statutory Workflow Engine** — enforces the routing matrix above; blocks invalid transitions server-side; records a mandatory reason on every return/revert.
3. **Case Rehearsal Sandbox** — the flagship differentiator (Section 4.1); isolated, full-case, read-only-baseline simulation.
4. **Deadline & SLA Intelligence** — per-stage ageing, durable scheduler, 90/60/30/15/7-day (or configured) alert ladder, catch-up recovery after a missed run.
5. **EVC Coordination** — eligibility queue, scheduling/rescheduling with reasons, geotagged visit confirmation, report-pending tracking (a completed visit is never conflated with an accepted report).
6. **Obligation & Undertaking Ledger** — submitted vs. verified states kept distinct; resubmission history preserved.
7. **Document Validity Registry** — issue/expiry tracking with bounded-confidence OCR suggestions, human-confirmed before anything is marked verified; full renewal lineage preserved.
8. **Multi-Channel Notification Service** — in-app, email, SMS, WhatsApp-style channels, each clearly marked **Sent / Pending / Failed / Simulated**, with deduplication keys so no case is double-alerted.
9. **Escalation Ladder** — unresolved alerts auto-escalate (Institution → RO → Regulation Bureau), with every level and recipient configurable and versioned.
10. **Role-Scoped Dashboards & Reports** — Institution / RO / EVC-SAC / Regulation Bureau views, all drilling into the same underlying filtered records (no dashboard-vs-export mismatch).
11. **Signed Audit Ledger** — every transition, verification, and rehearsal action logged, hash-chained and Ed25519-signed, with a one-click integrity check that names the exact tampered entry if verification fails.

---

## 8. USPs — What Makes This the Strongest Submission

### USP 1 — Full-Case Rehearsal Sandbox (not rule-only)
Hypothetical evidence or rule changes are tested against the **entire case state** — workflow stage eligibility, deadline recalculation, EVC eligibility, and the notifications that would fire — in one isolated, non-destructive operation. This directly exceeds the scope of the one competitor who built something similar.

### USP 2 — Statutory Routing as Governed, Versioned Data
Four distinct, named application routes (LOA / EOA-Standard / EOA-3-Year-Merit / Infra Modification) are configuration objects with an approval lifecycle (`Draft → Review → Approved → Published → Superseded`) — not scattered conditionals, and not silently editable by a live admin API.

### USP 3 — Dual-Confidence Evidence Model
Every extracted or declared fact carries its source and confidence explicitly in the schema and the UI — declared, OCR-suggested, and human-verified are three distinct states that are never silently merged.

### USP 4 — Signed, Not Just Hashed, Audit Trail
Asymmetric Ed25519 signing on top of the hash chain means tampering requires compromising a private key, not merely recomputing a hash — a materially stronger integrity guarantee than every competitor reviewed.

### USP 5 — Honesty as a Built-In UI State, Not Just a README Note
Every simulated channel, mock adapter, or demo-assumption value is labeled **in the interface itself** at the point a judge would see it — matching the discipline of the most rigorous competitor, applied consistently across every module, not just the integration layer.

### USP 6 — Human Authority Is Structural, Not Just Stated
No automated path exists anywhere in the schema or API for AI or a rule engine to independently approve, reject, or declare non-compliance — the boundary is enforced by what actions the data layer permits, not only by a policy statement.

---

## 9. Security & Compliance

- **Authentication:** Argon2id password hashing; mandatory TOTP 2FA for RO, EVC/SAC, and Regulation Bureau roles; short-lived access tokens with rotating refresh tokens.
- **Authorization:** Server-enforced RBAC with regional and institutional scope — never a frontend-only permission check.
- **Rehearsal isolation:** A dedicated PostgreSQL schema and least-privilege database role ensure simulation writes can never reach production tables, verified by an automated isolation check rather than asserted by documentation alone.
- **Documents:** AES-256-GCM encryption at rest, SHA-256 fingerprinting, MIME/size validation, served only via authenticated streaming — never a public or predictable URL.
- **Audit:** Append-only, hash-chained, Ed25519-signed ledger covering every transition, verification, rehearsal action, and rejected attempt.
- **Data isolation:** Institution- and region-scoped queries enforced server-side; cross-tenant access attempts are themselves logged as audit events.

---

## 10. What's Real vs. What's Simulated (stated up front)

| Capability | Status in Prototype | Path to Real |
|---|---|---|
| Workflow engine, routing matrix, deadline scheduler, obligation/document ledger, rehearsal sandbox, signed audit ledger | **Real** | Production-shaped already; hardening and load-testing remain |
| Bank / external financial confirmation (where applicable to linked compliance items) | **Simulated**, clearly badged | Requires formal inter-agency agreement |
| SMS / WhatsApp delivery | **Simulated outbox**, in-app and email channels functionally real | Requires an authorized NIC/Meta Business API account |
| AICTE existing-system integration (application identity, prior decisions) | **Mock adapter with synthetic records**, reconciliation logic real | Requires AICTE IT coordination and an integration contract |
| EVC GPS geotagging | **Simulated coordinate capture** in the prototype | Requires authorized device-level geofencing in a fielded mobile app |

Stating this plainly, in-app, is itself a competitive advantage — it is the single biggest credibility gap in most of the other five teams' submissions, where "production backend," "target architecture," or a live-sounding integration is implied without this boundary being visible to the user at the point of use.

---

## 11. Judge Demonstration Script (5 minutes)

1. **Statutory routing (60s):** Submit a New Institute application → show it load the LOA route automatically; submit an EOA application with NIRF/NBA/NAAC data → show it route to the 3-Year Merit track instead.
2. **Deficiency & resubmission (60s):** RO returns the LOA case with a recorded reason → institution corrects and resubmits → history preserved.
3. **Rehearsal sandbox — the differentiator (90s):** Officer opens the Rehearsal Sandbox on a borderline case, tests a hypothetical corrected lease date → system shows the full downstream effect: compliance rule flips, EVC eligibility becomes true, SLA deadline recalculates, and the notification that *would* fire is previewed — **none of it touches the real case** until the officer takes the real, audited action separately.
4. **EVC + escalation (60s):** Schedule an EVC, geotag the visit (simulated, badged), advance the demo clock → show the 15-day alert fire and an unacknowledged case auto-escalate from RO to Regulation Bureau.
5. **Audit integrity (30s):** Run "Verify Chain" → inject a tamper → show the system catch it and name the exact entry.

---

## 12. Measurable Outcomes We Will Actually Test

- Routing accuracy: correct route assigned per application category, across all four configured routes.
- Rehearsal fidelity: rehearsal-predicted outcome matches the real outcome once the same change is applied for real, across test cases.
- Deadline reliability: zero missed or duplicated alerts across a simulated multi-week time-jump run.
- Audit completeness: 100% of state-changing actions produce a verifiable, signed ledger entry.
- Separation of duties: zero successful self-approval attempts in adversarial testing.

No outcome is claimed without a stated, repeatable method of testing it — unlike headline turnaround-time or cost-saving figures asserted elsewhere without a baseline.

---

*This document is the proposed-solution overview for SIH26262. It supersedes our earlier NiyamSetu draft's Section 17.3 "optional, Phase 6" treatment of rehearsal — that capability is now core, scoped wider than any competing build, and demonstrated up front.*
