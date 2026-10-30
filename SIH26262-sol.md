# Proposed Solution — NiyamSetu
### Detailed Explanation, Problem Fit, Innovation & USP
**SIH26262 · AICTE, Ministry of Education**

---

## 1. The Problem, Precisely

AICTE's approval processing spans multiple application categories, scrutiny stages, Expert Visit Committees (EVCs), hearing/appellate committees, authorized officials, and time-bound institutional obligations — each governed by the **Approval Process Handbook (APH)**. Today, maintaining a reliable connection between all of this is difficult because:

1. **The route an application should take is not reliably connected to its category** — different application types (new institution, extension, merit-track extension, infrastructure change) follow genuinely different statutory paths.
2. **Ownership of the next action is unclear** — a pending case may not have a visibly accountable actor or deadline.
3. **Evidence and verification get conflated** — a submitted document is not the same as a reviewed and accepted one, but systems often treat them as equivalent.
4. **Deadlines are tracked manually** — maturity of undertakings, document validity, and EVC scheduling windows are easy to miss without durable, automated monitoring.
5. **There is no trustworthy history** — officials cannot always prove why a case is in its current state or who authorized a given action.
6. **Officers cannot safely test "what if"** — correcting a document or re-evaluating a rule today means acting on the real case; there is no safe way to check the consequence of a change before committing to it.

## 2. The Solution, In One Sentence

**NiyamSetu is a regulation-aware workflow-orchestration platform that routes every AICTE application through its correct, versioned statutory path, tracks every obligation and deadline automatically, and lets officers safely rehearse the full consequence of a hypothetical change before acting for real — all under a cryptographically signed audit trail.**

---

## 3. How It Addresses Every Element of the Problem Statement

| PS Requirement | NiyamSetu Mechanism |
|---|---|
| Configurable application routing by category | **Statutory Routing Matrix** — versioned `WorkflowDefinition` per category (LOA, EOA-Standard, EOA-3-Year-Merit, Infra Modification) |
| Automatic progression when conditions are met, with authorized reversion | **Workflow State Machine** — server-enforced guarded transitions, mandatory reason on every return |
| Role-based dashboards, pendency and ageing visibility | Role-scoped views for Institution / RO / EVC-SAC / Regulation Bureau, all drilling into the same underlying records |
| EVC eligibility, scheduling and completion monitoring | **EVC Coordination module** — eligibility queue, scheduling with reasons, geotag-ready visit confirmation, separate report-pending tracking |
| Time-bound institutional undertakings | **Obligation Ledger** — submitted vs. verified kept as distinct, auditable states |
| Document/lease validity monitoring and renewal | **Document Validity Registry** — expiry tracking, bounded-confidence OCR suggestions, full renewal lineage |
| Configurable reminders and escalation | **Deadline Scheduler + Escalation Ladder** — deduplicated, multi-channel, auto-escalating |
| Complete application and notification audit history | **Signed, hash-chained Audit Ledger** — every transition, verification, and rehearsal action logged |
| Secure document handling | AES-256-GCM encryption at rest, SHA-256 fingerprinting, authenticated access only |
| Integration with authorized existing systems | Explicit, swappable **Mock Adapter boundary** — real reconciliation logic, honestly labeled simulated data |

Every element of the official problem statement is addressed by name, not reinterpreted.

---

## 4. What Makes It Innovative — The USPs

### USP 1 — The Case Rehearsal Sandbox (full-case, not rule-only)
Most workflow tools, if they offer any "what-if" capability at all, test a single rule in isolation. NiyamSetu's Sandbox simulates the **entire case** — stage eligibility, deadline recalculation, EVC eligibility, and the exact notifications that would fire — against a database-isolated snapshot, so an officer sees the full downstream consequence of a hypothetical change in one step, with a provable guarantee that nothing touches the real case until a separate, audited real action is taken.

### USP 2 — Statutory Routing as Governed, Versioned Data
The four core application routes are not scattered conditionals; they are approved configuration objects with their own lifecycle (`Draft → Review → Approved → Published → Superseded`). A rule change creates a new version — it never silently rewrites how an in-flight case is being processed.

### USP 3 — Dual-Confidence Evidence Model
Every financial or procedural fact an institution submits is tracked alongside what was independently extracted or confirmed, with a visible confidence and source. The system never blends "what was declared" and "what was verified" into a single ambiguous field.

### USP 4 — Structural Separation Between Submission and Verification
An obligation or document can be *submitted* without being *verified* — these are distinct, trackable states in the schema itself, not a UI convention that could be bypassed.

### USP 5 — Signed, Not Just Hashed, Audit Trail
Every audit entry is hash-chained **and** Ed25519-signed. Tampering requires forging a private key, not merely recomputing a hash — a materially stronger non-repudiation guarantee than a plain hash chain provides.

### USP 6 — Honest Capability Boundary, Visible In-App
Every simulated channel (SMS, WhatsApp, a mock AICTE system sync, a geotag capture) is labeled as such at the point a user actually sees it — not buried in a README. This is a credibility feature: every claim the system makes can be defended under direct questioning.

### USP 7 — Human Authority Is Structural
No code path anywhere allows the rules engine, the scheduler, or any AI-assisted summarization to independently approve, reject, or declare statutory non-compliance. The boundary is enforced by what the data layer permits an automated process to write — not only stated as policy.

---

## 5. Why This Is Different From "Just a Digital Workflow Tracker"

A generic workflow tracker moves a ticket through stages. NiyamSetu is built around the idea that a **regulatory process's job is not just to move a case forward — it is to make every routing decision explainable, every deadline traceable to the event that triggered it, and every hypothetical change testable without risk before it becomes real.** That shift — from *tracking* to *provable, rehearsable governance* — is the core innovation.

---

## 6. Target Users & Their Experience

| User | What changes for them |
|---|---|
| **Institution** | Submits once per the correct category-specific route; sees exactly which obligations are pending vs. verified; gets proactive deadline reminders instead of discovering a lapse after the fact |
| **Regional Officer (RO)** | Reviews a pre-routed, pre-flagged queue; can rehearse the effect of a correction before committing to a real return/revert decision |
| **EVC / SAC Committee** | Sees only eligible, correctly-routed cases; visit and report status are tracked as distinct, unambiguous states |
| **Regulation Bureau** | Gets a cross-region, reconciled view where the dashboard and any exported report always agree, backed by a verifiable audit trail |

---

*For the underlying architecture and engineering detail behind these claims, see `SIH26262-Technical.md`. For feasibility, risk mitigation, and expected impact, see `SIH26262-Feasible.md`.*
