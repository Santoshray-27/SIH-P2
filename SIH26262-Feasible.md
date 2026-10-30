# Feasibility, Viability & Impact — NiyamSetu
### Regulation-Aware Workflow Orchestration for AICTE Approval Processing
**SIH26262 · AICTE, Ministry of Education**

---

## 1. Feasibility Assessment

### 1.1 Technical Feasibility
NiyamSetu is built entirely from mature, well-documented, open technology — React, FastAPI, PostgreSQL, Argon2id, Ed25519, AES-256-GCM. The system's flagship capability, the Case Rehearsal Sandbox, depends only on a standard PostgreSQL feature (schema-level roles and grants), not on any exotic infrastructure. Every "real" capability claimed (workflow engine, signed audit ledger, deadline scheduler, dual-confidence evidence model, rehearsal isolation) is demonstrable within the hackathon build window, because the stack was deliberately chosen to be **the smallest set of components that satisfies every PS requirement** — not the largest set that sounds impressive.

### 1.2 Operational Feasibility
The four statutory routes (LOA, EOA-Standard, EOA-3-Year-Merit, Infra Modification) mirror categories that already exist in AICTE's approval process, so adoption requires **digitizing and strengthening an existing process, not inventing a new one.** Role-scoped views mean each actor — Institution, RO, EVC/SAC, Regulation Bureau — only sees what is relevant to their function, minimizing training burden.

### 1.3 Economic Feasibility
The stack uses open-source components throughout — no licensing cost for the core system. Hosting a prototype or pilot-scale deployment is achievable on standard commodity cloud infrastructure. The largest future cost driver is not the software but **formal integration work** with external systems (AICTE's existing IT estate, an authorized SMS/WhatsApp business gateway) — correctly scoped as a negotiated, authorized follow-on activity rather than hidden inside the prototype's cost or claimed as already working.

### 1.4 Legal & Regulatory Feasibility
The system operates **within** AICTE's existing procedural authority rather than requiring new statutory power: it digitizes and strengthens accountability for a process AICTE already performs under the APH. Every point where the prototype cannot yet obtain authorization (live system integration, SMS/WhatsApp delivery, device-level geotagging) is named explicitly rather than assumed away, so the path to approval is clear rather than glossed over.

---

## 2. Viability — Why It Holds Up Under Scrutiny

| Dimension | Why NiyamSetu is viable, not just plausible |
|---|---|
| **Correctness** | Deterministic, versioned routing and rule evaluation — every routing decision and flag is explainable and reproducible, exactly what a regulatory process requires |
| **Trustworthiness** | A cryptographically signed, hash-chained audit ledger means the system's own claims about "what happened and when" are independently verifiable, not just logged |
| **Safety of experimentation** | The Rehearsal Sandbox lets officers test real consequences without any risk to a live case — a capability most competing approaches don't offer, and the one that does offer it only at the rule level, not the full case level |
| **Honesty under questioning** | Every simulated component is explicitly labeled in the UI itself — the system cannot be caught overstating what it does |
| **Low operational risk** | A lean, explainable stack with no unnecessary service dependencies means fewer failure points in a live demo or pilot deployment |
| **Alignment with existing process** | The routing matrix and obligation ledger mirror AICTE's actual categories and procedures, so adoption doesn't require reinventing institutional behavior |

---

## 3. Risks & Mitigation

| Risk | Mitigation |
|---|---|
| Statutory routing is misconfigured relative to the real APH | Every route is labeled a prototype assumption pending authorized clause mapping; routes are versioned so a correction never silently rewrites an in-flight case |
| OCR misreads a document field | Every extracted field carries a confidence score and is shown side-by-side with the declared value — a human always makes the final call |
| A rehearsal session accidentally affects a real case | Prevented structurally — the rehearsal schema is a separate, least-privilege-enforced database boundary, verified by an automated isolation check, not just application-level logic |
| Institutions resist a new mandatory digital process | The routing and obligation model mirrors the existing process; the system reduces institution effort (status visibility, proactive reminders) rather than adding new burden |
| Future SMS/WhatsApp integration is delayed or denied | The system already functions fully on in-app and email channels; external channels are an enhancement, not a dependency for core operation |
| Audit ledger storage grows large over a multi-year case lifecycle | Append-only ledger entries are lightweight (hash + signature + metadata); standard archival/partitioning strategies apply without touching verification logic |

---

## 4. Impact & Benefits

### 4.1 For AICTE's Regulation Bureau and Regional Offices
- A single, always-current, correctly-routed view of every application, replacing fragmented manual coordination across stages and committees.
- A safe way to test the consequence of a correction or a policy change before committing to it for real — reducing the risk of a rushed or reversible-only-with-difficulty decision.
- A verifiable, non-repudiable audit trail for every routing decision and escalation — strengthening accountability where it matters most.
- Proactive visibility into approaching deadlines and EVC scheduling windows, reducing the risk of cases stalling unnoticed.

### 4.2 For Technical Institutions
- A single, category-correct submission path instead of ambiguity about which process applies.
- Clear, real-time visibility into what has been verified versus what is still pending — no ambiguous status.
- Advance reminders before obligation and document-renewal deadlines, reducing the risk of inadvertent non-compliance.

### 4.3 For the Regulatory Process as a Whole
- Faster, more defensible scrutiny through automated, pre-screened obligation and document checks.
- A tamper-evident record suitable for internal review or external audit.
- A foundation that can extend — once formally authorized — to live external integrations, without re-architecting anything already built.

### 4.4 Measurable Indicators (testable directly in the prototype)
- Routing accuracy across all four configured statutory categories.
- Rehearsal fidelity: the rehearsal-predicted outcome matches the real outcome once the same change is applied for real.
- Deadline reliability: zero missed or duplicated alerts across a simulated multi-week time-jump run.
- Audit completeness: 100% of state-changing actions produce a verifiable, signed ledger entry.
- Separation-of-duties integrity: zero successful self-approval attempts under adversarial testing.

No outcome is asserted without a stated method of measurement — every figure above is something the prototype can actually be tested against, not a projected or assumed number.

---

## 5. Scalability Outlook

The relational data model (institution → application → routing/lineage) scales horizontally by region without schema changes. The scheduler and audit ledger are stateless per-transaction, so additional load is handled by scaling the application layer, not by redesigning core logic. The Rehearsal Sandbox's isolation mechanism (a separate schema and database role) scales the same way — it adds no cross-case coupling as case volume grows. Moving from prototype to a multi-region, AICTE-wide deployment is an infrastructure change, not a rebuild.

---

*This document should be read alongside `SIH26262-sol.md` (what the solution does and why it's novel) and `SIH26262-Technical.md` (how it is built).*
