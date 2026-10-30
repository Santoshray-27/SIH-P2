<p align="center">
  <img src="https://img.shields.io/badge/SIH-26262-blue?style=for-the-badge" alt="SIH Problem ID" />
  <img src="https://img.shields.io/badge/AICTE-Ministry_of_Education-green?style=for-the-badge" alt="AICTE" />
  <img src="https://img.shields.io/badge/Status-Prototype-orange?style=for-the-badge" alt="Status" />
</p>

<h1 align="center">🌉 NiyamSetu</h1>
<h3 align="center">Regulation-Aware Workflow Orchestration for AICTE Approval Processing</h3>

<p align="center">
  <em>नियम + सेतु — the bridge between a regulatory rule and the action it governs.</em><br/>
  <strong>Every case on the right path. Every obligation on time. Every decision provably traceable.</strong>
</p>

---

## 📌 Problem Statement

AICTE's approval processing spans multiple application categories, scrutiny stages, Expert Visit Committees (EVCs), hearing committees, authorized officials, and time-bound institutional obligations — each governed by the **Approval Process Handbook (APH)**.

Current pain points:

- ❌ Application routing is not reliably connected to its category
- ❌ Ownership of the next action is unclear — no visible accountability or deadlines
- ❌ Evidence submission ≠ verification, but systems treat them as equivalent
- ❌ Deadlines are tracked manually — easy to miss
- ❌ No trustworthy, tamper-proof history of decisions
- ❌ No safe way to test "what if" scenarios without affecting real cases

---

## 💡 Our Solution

**NiyamSetu** is a regulation-aware workflow-orchestration platform that:

> Routes every AICTE application through its correct, versioned statutory path, tracks every obligation and deadline automatically, and lets officers safely rehearse the full consequence of a hypothetical change before acting for real — all under a cryptographically signed audit trail.

---

## 🏗️ Architecture

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
                     │ FastAPI (Python, async)             │
                     │ ├─ Case Registry & Routing Engine   │
                     │ ├─ Statutory Workflow State Machine │
                     │ ├─ Deadline & Escalation Scheduler  │
                     │ ├─ Obligation & Document Ledger     │
                     │ ├─ Evidence Extraction (OCR)        │
                     │ ├─ Case Rehearsal Sandbox Engine     │
                     │ └─ Signed Audit Ledger Service       │
                     └───────┬───────────────┬───────────────┘
                             │               │
               ┌─────────────▼──┐   ┌───────▼───────────────┐
               │  PostgreSQL      │   │ Rehearsal Schema        │
               │  (operational    │   │ (isolated, least-       │
               │  system of       │   │ privilege DB role —     │
               │  record)         │   │ never touches prod)     │
               └──────────────────┘   └──────────────────────────┘
                             │
                     ┌───────▼────────────┐
                     │ Encrypted Document   │
                     │ Store (AES-256-GCM,  │
                     │ SHA-256 fingerprint)  │
                     └──────────────────────┘
```

---

## ⚙️ Tech Stack

| Layer | Technology | Rationale |
|---|---|---|
| **Frontend** | React 18 + TypeScript + Vite + Tailwind | Typed, role-scoped views for 4 user types |
| **Backend** | FastAPI (Python 3.11+, async) | Live OpenAPI docs double as a judge-facing technical artifact |
| **Database** | PostgreSQL 15+ | ACID integrity; role-based schema isolation for the rehearsal sandbox |
| **ORM** | SQLAlchemy 2.x + Alembic | Versioned, reviewable schema evolution |
| **Auth** | Argon2id + JWT + TOTP 2FA | Strong credential hygiene with minimal operational surface |
| **Audit** | Hash-chained + Ed25519-signed ledger | Tampering requires forging a private key, not just recomputing a hash |
| **Scheduler** | In-process durable worker (idempotency-keyed) | Deduplicated alerts without Redis/Celery dependency |

---

## 🌟 Key Features & USPs

### 🧪 USP 1 — Full-Case Rehearsal Sandbox
Unlike rule-only "what-if" checkers, our sandbox simulates the **entire case state** — stage eligibility, deadline recalculation, EVC eligibility, and notifications — in one isolated, non-destructive operation. Officers see the full downstream consequence of a hypothetical change before taking action.

### 📋 USP 2 — Statutory Routing as Versioned Data
Four distinct application routes (LOA / EOA-Standard / EOA-3-Year-Merit / Infra Modification) are governed configuration objects with an approval lifecycle (`Draft → Review → Approved → Published → Superseded`). A rule change creates a new version — it never silently rewrites in-flight cases.

### 🔍 USP 3 — Dual-Confidence Evidence Model
Every fact carries its source and confidence explicitly: **declared**, **OCR-suggested**, and **human-verified** are three distinct states, never merged.

### 🔐 USP 4 — Signed Audit Trail
Every entry is hash-chained **and** Ed25519-signed. One-click chain integrity verification pinpoints the exact entry if tampering is detected.

### 🏷️ USP 5 — Honesty as a UI State
Every simulated component (SMS, WhatsApp, mock adapters) is labeled as such in the interface itself — not buried in documentation.

### 🛡️ USP 6 — Structural Human Authority
No automated path permits AI or rule engines to independently approve, reject, or declare non-compliance. The boundary is enforced at the data layer.

---

## 📊 Statutory Routing Matrix

```
NEW INSTITUTE (LOA)
  SUBMITTED → SCRUTINY → [RE-SCRUTINY] → EVC → [SAC/SHC] → LOA DECISION

EOA STANDARD
  SUBMITTED → SCRUTINY → EOA DECISION
  (routes to EVC only if deficiency/breach is flagged)

EOA 3-YEAR MERIT TRACK
  SUBMITTED → AUTOMATED ELIGIBILITY CHECK → EOA DECISION

INFRASTRUCTURE MODIFICATION
  SUBMITTED → EVC → [SAC/SHC] → DECISION
```

> Routes are published, immutable configuration objects. A case keeps the version it was created under.

---

## 👥 Target Users

| Role | What NiyamSetu Changes |
|---|---|
| **Institution** | Single category-correct submission path; clear visibility into verified vs. pending status; proactive deadline reminders |
| **Regional Officer (RO)** | Pre-routed, pre-flagged queue; rehearse corrections before committing; manage EVC scheduling |
| **EVC / SAC Committee** | Only eligible, correctly-routed cases; distinct visit vs. report tracking |
| **Regulation Bureau** | Cross-region reconciled view; verifiable audit trail; versioned rule management |

---

## 🔒 Security

- **Authentication:** Argon2id hashing, mandatory TOTP 2FA for officers, short-lived JWTs with rotating refresh tokens
- **Authorization:** Server-enforced RBAC with regional/institutional scope
- **Rehearsal Isolation:** Dedicated PostgreSQL schema + least-privilege DB role, verified by automated isolation checks
- **Documents:** AES-256-GCM at rest, SHA-256 fingerprinting, authenticated-only access
- **Audit:** Append-only, hash-chained, Ed25519-signed — `AuditEntry` and `TransitionRecord` tables have no UPDATE/DELETE grants
- **Tenant Isolation:** Cross-tenant access attempts are logged as audit events

---

## 📄 Documentation

| Document | Description |
|---|---|
| [`SIH26262-sol.md`](./SIH26262-sol.md) | Solution overview — problem fit, innovation & USPs |
| [`SIH26262-solution-2.md`](./SIH26262-solution-2.md) | Revised solution with competitive analysis & design principles |
| [`SIH26262-Technical.md`](./SIH26262-Technical.md) | Technical architecture, data model & core algorithms |
| [`SIH26262-Feasible.md`](./SIH26262-Feasible.md) | Feasibility assessment, risks, impact & scalability |
| [`SIH26262-frontend.md`](./SIH26262-frontend.md) | Complete 32-page frontend blueprint & UX specification |
| [`NiyamSetu (4).html`](<./NiyamSetu (4).html>) | Interactive UI prototype |

---

## 📏 Measurable Outcomes

- ✅ **Routing accuracy** — correct route assigned per application category across all 4 routes
- ✅ **Rehearsal fidelity** — predicted outcome matches real outcome when the same change is applied
- ✅ **Deadline reliability** — zero missed or duplicated alerts across simulated time-jump runs
- ✅ **Audit completeness** — 100% of state-changing actions produce a verifiable, signed ledger entry
- ✅ **Separation of duties** — zero successful self-approval attempts under adversarial testing

---

## ⚠️ What's Real vs. Simulated

| Capability | Status | Path to Production |
|---|---|---|
| Workflow engine, routing, deadlines, audit ledger, rehearsal sandbox | ✅ **Real** | Hardening & load-testing |
| SMS / WhatsApp delivery | 🟡 **Simulated outbox** | Requires authorized NIC/Meta Business API |
| AICTE system integration | 🟡 **Mock adapter** (reconciliation logic is real) | Requires AICTE IT coordination |
| EVC GPS geotagging | 🟡 **Simulated** | Requires fielded mobile app |
| Bank / financial confirmation | 🟡 **Simulated** | Requires inter-agency agreement |

> Every simulated component is labeled in the UI itself — transparency is a feature, not a footnote.

---

## 🚀 Getting Started

```bash
# Clone the repository
git clone https://github.com/Satwik0212/NiyamSetu-SIH26262.git
cd NiyamSetu-SIH26262

# Open the prototype
# Open NiyamSetu (4).html in your browser to explore the UI prototype

# Review the documentation
# Start with SIH26262-sol.md for the solution overview
```

---

## 🎬 Demo Script (5 min)

1. **Statutory Routing (60s)** — Submit a New Institute app → auto-routes to LOA track; submit an EOA with merit data → routes to 3-Year Merit track
2. **Deficiency & Resubmission (60s)** — RO returns case with reason → institution corrects → history preserved
3. **Rehearsal Sandbox (90s)** — Test hypothetical corrected lease date → see full downstream effect (compliance flips, EVC eligibility, SLA recalculates, notification preview) — **nothing touches the real case**
4. **EVC + Escalation (60s)** — Schedule EVC, geotag visit, advance clock → alerts fire, unacknowledged case auto-escalates
5. **Audit Integrity (30s)** — Run "Verify Chain" → inject tamper → system catches it, names exact entry

---

## 📜 License

This project was developed for **Smart India Hackathon 2026** (Problem Statement SIH26262) submitted to **AICTE, Ministry of Education, Government of India**.

---

<p align="center">
  <strong>Built with ❤️ for Smart India Hackathon 2026</strong><br/>
  <em>NiyamSetu — Provable, rehearsable governance for India's technical education approvals.</em>
</p>

