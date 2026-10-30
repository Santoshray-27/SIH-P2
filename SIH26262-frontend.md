# Frontend Blueprint — NiyamSetu
### Complete Page Inventory & UX Specification
**SIH26262 · AICTE, Ministry of Education**

---

## 0. Scope

This is the full product blueprint — every page a complete, multi-role deployment would need. Section 9 flags the subset that actually matters for the 5-minute judge demo, so the build can be prioritized without losing sight of the whole picture.

**Total: 32 unique pages** across 4 roles (Institution, Regional Officer, EVC/SAC Committee, Regulation Bureau) plus a public zone and a shared-template layer.

---

## 1. Visual Identity & Design Principles

| Element | Direction |
|---|---|
| **Tone** | Government-grade credibility with modern SaaS clarity — never playful, never sterile |
| **Color system** | Deep indigo/navy as the primary (trust, authority) + a single accent teal for positive/active states + amber for pending/warning + red reserved *only* for overdue/high-severity — color must never be the only signal (pair with icon + label) |
| **Typography** | One humanist sans for UI text (e.g., Inter/Plus Jakarta Sans), one monospace for IDs, hashes, and audit entries — never more than two families |
| **Layout grammar** | Card-based content blocks, consistent 12-column grid, one page-level action button maximum per view (secondary actions go in an overflow menu) |
| **Status language** | Every status badge uses the same verb-noun pattern system-wide (e.g., "Awaiting RO", "Verified", "Overdue") — never a bare color chip with no label |
| **Honesty marker** | A small, consistent "Simulated" tag (not hidden in a tooltip) on every element backed by a mock adapter — SMS/WhatsApp previews, bank confirmations, geotags |
| **Motion** | Minimal — state transitions and the Rehearsal Sandbox diff view are the only places animation carries meaning |
| **Accessibility** | WCAG AA contrast minimum, full keyboard navigation, visible focus states, no information conveyed by color alone |

---

## 2. Information Architecture

```
Public Zone (unauthenticated)
   └── Landing → Login → 2FA → [role-aware redirect]

Authenticated Shell (persistent top bar + role-scoped side nav)
   ├── Institution
   ├── Regional Officer (RO)
   ├── EVC / SAC Committee
   └── Regulation Bureau (HQ)

Shared Layer (same template, role-scoped data, reachable from any role)
   ├── Notifications
   ├── Profile & Security Settings
   ├── Help / Demo Guide
   └── Global Search
```

Each role sees only its own side-navigation items — there is one shell, not four separate apps, so switching the demo persona changes *content*, not the whole interface skeleton.

---

## 3. Full Page Inventory

| # | Page | Route | Role(s) | Purpose |
|---|---|---|---|---|
| 1 | Landing Page | `/` | Public | Explain NiyamSetu, problem framing, CTA to log in |
| 2 | Login | `/login` | Public | Credential entry + demo persona switcher |
| 3 | 2FA Verification | `/login/verify` | Public → RO/EVC-SAC/Bureau | TOTP code entry |
| 4 | Forgot Password | `/login/forgot` | Public | Reset flow |
| 5 | Transparency Page | `/transparency` | Public | "What's Real vs. Simulated" — the honesty boundary, stated up front |
| 6 | 404 / Not Found | `/404` | Public | Graceful dead-end |
| 7 | Institution Dashboard | `/institution` | Institution | Active applications, obligations due, document expiry, notifications |
| 8 | New Application Wizard | `/institution/applications/new` | Institution | Category selection → details → document upload |
| 9 | Application Detail / Case Timeline | `/institution/applications/:id` | Institution | Stage, deficiencies, required actions, history |
| 10 | Obligations & Compliance | `/institution/obligations` | Institution | List obligations, submit evidence |
| 11 | Document Vault | `/institution/documents` | Institution | Upload, track expiry, renew |
| 12 | RO Dashboard | `/ro` | Regional Officer | Region-scoped pendency, ageing, workload |
| 13 | Case Queue (Scrutiny Worklist) | `/ro/queue` | Regional Officer | Filterable list of assigned/unassigned cases |
| 14 | Case Scrutiny Workspace | `/ro/applications/:id` | Regional Officer | Document verification, deficiency issuance, transitions |
| 15 | Case Rehearsal Sandbox | `/ro/applications/:id/rehearsal` | Regional Officer | The flagship feature — full-case hypothetical simulation |
| 16 | EVC Scheduling & Coordination | `/ro/evc` | Regional Officer | Eligibility queue, scheduling, rescheduling |
| 17 | Obligation Review Queue | `/ro/obligations` | Regional Officer | Verify institution-submitted evidence |
| 18 | Document Renewal Review Queue | `/ro/documents` | Regional Officer | Review renewed documents before promotion to current |
| 19 | Escalation Center | `/ro/escalations` | Regional Officer | Unresolved escalations needing action |
| 20 | EVC/SAC Dashboard | `/evc` | EVC/SAC Committee | Assigned cases, schedule |
| 21 | EVC Visit & Report Workspace | `/evc/visits/:id` | EVC/SAC Committee | Visit confirmation, geotag, report submission |
| 22 | Bureau National Dashboard | `/bureau` | Regulation Bureau | Cross-region KPIs, charts, trends |
| 23 | Regional Workload & Bottleneck Reports | `/bureau/reports` | Regulation Bureau | Comparative region/officer analytics |
| 24 | Rules & Workflow Configuration | `/bureau/rules` | Regulation Bureau | Routing matrix, deadline policy, escalation ladder — versioned admin |
| 25 | Audit Trail Explorer | `/bureau/audit` | Regulation Bureau | Searchable ledger + one-click chain-integrity verification |
| 26 | User & Role Management | `/bureau/users` | Regulation Bureau | Provision accounts, assign roles/regions |
| 27 | Reports & Export Center | `/bureau/exports` | Regulation Bureau | Statutory report generation, CSV export |
| 28 | Notification Center | `/notifications` | All (shared template) | Channel-filterable alert history |
| 29 | Profile & Security Settings | `/settings` | All (shared template) | Password, 2FA setup, notification preferences |
| 30 | Help / Demo Guide | `/help` | All (shared template) | In-app walkthrough — doubles as the judge-facing demo script |
| 31 | Global Search Results | `/search` | All (shared template) | Cross-entity search (case ID, institution, document) |
| 32 | Session/Access Denied | `/403` | All | Scope-violation feedback (e.g., cross-region access attempt) |

---

## 4. Public Zone — Detail

### 4.1 Landing Page (`/`)
- Hero: product name, tagline, one-line problem statement
- "How it works" 4-step visual (Submit → Route → Verify → Decide)
- Honesty strip: a visible banner linking to the Transparency Page — set the tone before anyone logs in
- Role explainer cards (Institution / RO / EVC-SAC / Bureau) with a one-line description each
- Footer: prototype disclaimer, synthetic-data notice

### 4.2 Login (`/login`)
- Email/password fields
- **Demo persona switcher** — one-click buttons for each seeded role (judge-critical: no manual credential typing needed mid-demo)
- "Forgot password" link
- Prototype banner: "All accounts and data are synthetic"

### 4.3 2FA Verification (`/login/verify`)
- 6-digit TOTP input, auto-advancing fields
- "Use a backup code" fallback link
- Shown only for RO / EVC-SAC / Bureau roles (Institution uses password-only in the prototype, labeled as a demo simplification)

### 4.4 Transparency Page (`/transparency`)
- A direct rendering of the Real-vs-Simulated table from the solution document
- This page is linked from the landing page, the login page footer, and every "Simulated" badge in the app — one authoritative place a judge can check any claim

### 4.5 404 / 403
- Minimal, on-brand, a single "Return to dashboard" action — no dead ends

---

## 5. Institution Pages — Detail

### 5.1 Institution Dashboard (`/institution`)
- KPI row: Active Applications, Obligations Due Soon, Documents Expiring, Unread Notifications
- "Needs Your Action" list (deficiencies to correct, evidence to submit) — sorted by urgency
- Recent activity feed

### 5.2 New Application Wizard (`/institution/applications/new`)
- Step 1: Category selection (LOA / EOA-Standard / EOA-3-Year-Merit / Infra Modification) — selecting a category immediately previews which statutory route will apply
- Step 2: Application details form
- Step 3: Document upload (drag-and-drop, per-document type slots)
- Step 4: Review & submit — shows the pinned workflow version that will govern this case

### 5.3 Application Detail / Case Timeline (`/institution/applications/:id`)
- Horizontal stage stepper (current stage highlighted, future stages greyed)
- Deadline countdown widget for the current stage
- Deficiency panel (if returned) with the officer's reason and a "Resubmit" action
- Full case timeline (read-only to this role)

### 5.4 Obligations & Compliance (`/institution/obligations`)
- Table: obligation, due date, status (Awaiting / Submitted / Under Review / Verified / Returned)
- Evidence upload modal per obligation
- Resubmission history inline

### 5.5 Document Vault (`/institution/documents`)
- Grid/list of documents with expiry countdown badges (color-graded by urgency)
- "Renew" action opens an upload flow linked to the expiring document (lineage preserved)
- Version history per document

---

## 6. Regional Officer (RO) Pages — Detail

### 6.1 RO Dashboard (`/ro`)
- Region-scoped KPI row: Pending, Overdue, EVC-Eligible, Obligations Awaiting Review
- Workload-by-officer mini chart
- "Oldest cases first" priority list

### 6.2 Case Queue (`/ro/queue`)
- Filterable/sortable table: category, stage, ageing, deadline, assigned officer
- Bulk-assign action (admin-permitted)
- Saved filter views

### 6.3 Case Scrutiny Workspace (`/ro/applications/:id`)
- Split view: document viewer (left) + declared-vs-extracted field comparison panel (right)
- Transition action bar (Advance / Return with mandatory reason / Flag for EVC)
- "Open in Rehearsal Sandbox" button — the bridge into the flagship feature

### 6.4 Case Rehearsal Sandbox (`/ro/applications/:id/rehearsal`) — Flagship Page
- Baseline panel (current real case state, read-only) side-by-side with a hypothetical-edit panel
- Officer edits a field (e.g., corrected lease date) in the hypothetical panel only
- "Run Rehearsal" triggers the isolated simulation
- **Diff view**: rule outcome changes, stage-eligibility changes, deadline recalculation, notification preview — each change highlighted with before/after
- Persistent banner: "Simulation only — production_modified: false" with a link to how isolation is enforced (ties back to Technical.md's database-role guarantee)
- "Discard" or "Apply for real" (the latter routes back to the real Scrutiny Workspace as a normal, audited action — never applies directly from the sandbox)

### 6.5 EVC Scheduling & Coordination (`/ro/evc`)
- Eligibility queue (cases that became EVC-eligible)
- Calendar/list view toggle for scheduling
- Reschedule action with mandatory reason

### 6.6 Obligation Review Queue (`/ro/obligations`)
- Queue of submitted-but-unverified obligations
- Accept / Return-for-correction actions with reason field

### 6.7 Document Renewal Review Queue (`/ro/documents`)
- Side-by-side: expiring document vs. newly submitted renewal
- Accept (promotes to current, preserves lineage) / Reject

### 6.8 Escalation Center (`/ro/escalations`)
- List of unresolved escalations routed to this officer
- Acknowledge / resolve actions, each writing an audit entry

---

## 7. EVC/SAC Committee Pages — Detail

### 7.1 EVC/SAC Dashboard (`/evc`)
- Assigned cases list with scheduled dates
- Upcoming visits this week highlighted

### 7.2 EVC Visit & Report Workspace (`/evc/visits/:id`)
- Visit confirmation action with simulated geotag capture (clearly badged)
- Inspection notes form
- Report submission (file + structured findings)
- Status progression: Scheduled → Confirmed → Completed → Report Submitted → Report Reviewed (a completed visit is visually distinct from an accepted report)

---

## 8. Regulation Bureau (HQ) Pages — Detail

### 8.1 Bureau National Dashboard (`/bureau`)
- Cross-region KPI strip + stage-wise pendency funnel chart
- Regional workload grouped bar chart
- Escalation volume trend

### 8.2 Regional Workload & Bottleneck Reports (`/bureau/reports`)
- Drill-down charts (click a bar → filtered case list, same filter logic as the dashboard — never a mismatched count)

### 8.3 Rules & Workflow Configuration (`/bureau/rules`)
- List of workflow definitions and deadline policies, each showing version, status (Draft/Review/Approved/Published/Superseded), effective date
- "Preview impact" before publishing a new version
- Publishing requires an explicit approval step — never a silent live edit

### 8.4 Audit Trail Explorer (`/bureau/audit`)
- Searchable, filterable ledger (actor, action, case, date range)
- "Verify Chain Integrity" button — recomputes hashes/signatures and names the exact entry if tampering is detected

### 8.5 User & Role Management (`/bureau/users`)
- Table of accounts, role, region, 2FA status
- Invite/deactivate actions

### 8.6 Reports & Export Center (`/bureau/exports`)
- Pre-built statutory report templates
- CSV export with the same scope/filter logic as every dashboard

---

## 9. Shared Pages — Detail

| Page | Notes |
|---|---|
| **Notification Center** | One template, role-scoped data; filter by channel (In-App/Email/SMS/WhatsApp) and delivery status (Sent/Pending/Failed/Simulated) |
| **Profile & Security Settings** | Password change, 2FA enrollment/QR, notification channel preferences |
| **Help / Demo Guide** | Doubles as the in-app version of the judge demo script — genuinely useful, not filler |
| **Global Search** | Single search bar in the top nav, results grouped by entity type (Case / Institution / Document / Obligation) |

---

## 10. Navigation Shell

- **Top bar** (persistent): product mark, global search, notification bell (badge count), demo persona switcher (prototype-only, removable for production), profile menu
- **Side nav** (role-scoped): changes per role, never more than 7 items — matches the page inventory above per role
- **Breadcrumbs** on every detail page (`Bureau / Audit Trail / Case APP-2026-0014`)
- **Status chips** are the one UI element repeated everywhere — same component, same color/label mapping, used on dashboards, queues, and detail pages alike

---

## 11. Core Reusable Components

| Component | Used on |
|---|---|
| `StatusBadge` | Everywhere — single source of truth for stage/obligation/document status colors and labels |
| `SimulatedTag` | Any element backed by a mock adapter — links to the Transparency Page |
| `DeadlineCountdown` | Case detail, obligations, document vault, dashboards |
| `WorkflowStepper` | Application detail, new-application wizard |
| `EvidenceComparisonPanel` | Scrutiny workspace, document renewal review |
| `RehearsalDiffViewer` | Rehearsal Sandbox — the most custom component in the system |
| `AuditTimeline` | Case detail, Audit Trail Explorer |
| `DataTable` (filterable, sortable, bulk-action capable) | Queues, user management, reports |
| `FileUploadDropzone` | Application wizard, document vault, evidence submission |
| `RoleSwitcher` (demo-only) | Login page |
| `ChartCard` (wraps Recharts-style visualizations) | Both dashboards, reports |

---

## 12. States Every Data Page Must Handle

- **Loading** — skeleton cards, never a blank screen
- **Empty** — a specific, helpful empty state per page (e.g., "No obligations due — you're caught up" rather than a generic "No data")
- **Error** — a retry affordance, never a raw stack trace
- **Scope-denied** — redirect to `/403` with a clear, non-alarming explanation

---

## 13. Demo-Critical Subset (if time only allows a priority build)

For the 5-minute judge walkthrough, these 11 pages carry the whole story — build these first and to full polish:

`Login → Institution Dashboard → New Application Wizard → RO Dashboard → Case Queue → Case Scrutiny Workspace → Case Rehearsal Sandbox → EVC Scheduling → Audit Trail Explorer → Bureau National Dashboard → Transparency Page`

Everything else in this document is the full product vision — valuable to show judges *exists on paper* (this document itself is evidence of that), but not all of it needs working code for the demo to land.
