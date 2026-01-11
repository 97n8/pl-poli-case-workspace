# VAULT Module Charters v1.0

**Quick Overview of All Modules**

---

## VAULT PRR™ — Public Records Requests

**Governing Law:** M.G.L. c. 66, §10 (Uniform Public Records Law)

**What it does:** Manages public records requests from intake through assembly, review, and delivery with statutory deadline enforcement.

**Why it matters:** Public records requests are high-legal-risk. Missing T10 (acknowledgment), T25 (completion/extension), or T90 (appeal window) has real consequences. VAULT PRR enforces these automatically.

**Key features:**
- T10 deadline enforcement (initial response)
- T25 gate (municipal production limit or extension required)
- T90 appeal window tracking
- Fee waiver automation (if T10 missed)
- Exemption logging and version control
- Complete audit trail for legal discovery

**Operational scale:** 10-200 requests/year (typical small-to-mid municipality)

**Complexity:** HIGH (strict statute, multiple exemption types, appeal potential)

**Status:** Fully specified (reference implementation) — ready to implement

---

## VAULT CLERK™ — Licensing & Permitting

**Governing Law:** M.G.L. c. 101 (varies by license type: Contractor, Plumbing, Electrical, Food Service, etc.)

**What it does:** Manages license applications from intake through inspection (if required), approval/denial, and issuance with statutory deadline enforcement.

**Why it matters:** License deadlines vary by type (10-30 days). Missed deadlines trigger automatic approval (deemed approval). Denials must be documented and locked. Appeals have 30-day windows.

**Key features:**
- License-type-specific decision trees (Contractor, Plumbing, Electrical, Food Service, Building Permit)
- Inspection verification gates
- Deemed approval enforcement (automatic approval if deadline missed)
- Approval/denial letter locking
- T90 appeal window tracking
- Renewal tracking (deferred to v1.1)

**Operational scale:** 50-500 applications/year

**Complexity:** MEDIUM-HIGH (multiple license types, inspection coordination, deemed approval edge cases)

**Status:** Fully specified — ready to implement

---

## VAULT FISCAL™ — Financial & Procurement

**Governing Law:** M.G.L. c. 41, §23 (Municipal Finance); M.G.L. c. 149 (Procurement); M.G.L. c. 30B (Uniform Procurement)

**What it does:** Manages invoices from receipt through 3-way match (invoice/PO/receipt), budget checking, approval routing, and payment with segregation of duties enforcement.

**Why it matters:** Financial controls are audit-critical. 3-way match is not optional. Approval authority varies by amount. Overspending must be prevented structurally. Payment records must be immutable.

**Key features:**
- 3-way match enforcement (invoice = PO = receipt)
- Budget availability checking
- Approval authority routing by amount ($0-$1K, $1-5K, $5-25K, $25K+)
- Segregation of duties (receiver ≠ approver ≠ payer)
- Vendor authorization verification
- Duplicate payment detection
- Payment method tracking (check, ACH, card)
- GL posting immutability
- Variance explanation and resolution

**Operational scale:** 500-5,000 invoices/year

**Complexity:** HIGH (multiple gates, financial integration, segregation of duties)

**Status:** Fully specified — ready to implement

---

## VAULT TIME™ — Timesheets & Payroll

**Governing Law:** M.G.L. c. 149 (Labor law, minimum wage, overtime); Local HR Policy

**What it does:** Manages timesheet entry, supervisor approval, and payroll posting with overtime routing and LOCKED records (cannot edit after payroll posting).

**Why it matters:** Timesheets are wage records. Overtime has budget impact and requires authorization. Once posted to payroll, timesheets cannot be altered (compliance requirement). Corrections must create new versions.

**Key features:**
- Weekly entry deadline enforcement (Friday EOD)
- Supervisor approval gate
- Overtime authorization routing (40h regular, 40-48h Finance approval, 48+ Director approval)
- Project/GL allocation tracking
- Budget checking for overtime
- Payroll posting LOCK (immutable after posting)
- Version chains for corrections
- No edits after posting

**Operational scale:** 10-200 employees, weekly entries

**Complexity:** MEDIUM (soft SLAs, straightforward state machine, payroll integration)

**Status:** Fully specified — ready to implement

---

## VAULT FIX™ — Maintenance & Work Orders

**Governing Law:** M.G.L. c. 30, §39K (Capital Asset Management); Local Facilities Policy

**What it does:** Manages work orders from intake through triage, scheduling, in-progress work, verification, and closure with SLA tracking and completion verification.

**Why it matters:** Maintenance creates institutional records of capital assets and asset care. Work must be verified before closure. SLAs ensure response times are tracked (not enforced hard, but tracked for accountability).

**Key features:**
- Priority-based SLA assignment (CRITICAL, HIGH, NORMAL, LOW with response/completion targets)
- Skill-based assignment routing
- Technician/contractor assignment
- In-progress work tracking
- Completion verification gates (pass/fail)
- SLA compliance monitoring (soft targets)
- Cost tracking and reconciliation
- Capital asset linkage

**Operational scale:** 100-1,000 work orders/year

**Complexity:** MEDIUM (straightforward state machine, SLA tracking, verification gates)

**Status:** Fully specified — ready to implement

---

## VAULT ONBOARD™ — Employee Onboarding

**Governing Law:** M.G.L. c. 149 (Labor law); Local HR Policy

**What it does:** Manages employee onboarding from offer acceptance through pre-start, Day 1, systems setup, training, and manager sign-off with mandatory task sequencing and training LOCK.

**Why it matters:** Onboarding is knowledge transfer. If it lives in one person's head, you lose it when they leave. VAULT ONBOARD makes onboarding explicit, documented, and locked so new employees get the same experience every time.

**Key features:**
- Background check hard gate (must clear before Day 1)
- Pre-start task sequencing (workspace, hardware, IT prep)
- Day 1 task checklist (I-9, badge, welcome, introduction)
- Systems access verification (email, network, payroll systems)
- Policy training LOCK (training records cannot be edited after completion)
- Department training LOCK
- Manager sign-off LOCK (final gate before close)
- No task can be skipped
- Task dependencies enforced

**Operational scale:** 10-50 new employees/year

**Complexity:** LOW (simplest module, no deadline enforcement, just task sequencing)

**Status:** Fully specified — ready to implement

---

## Module Independence (v1.0)

**Important:** In VAULT v1.0, modules are intentionally independent.

**What this means:**
- One work unit (CASE) = one module
- Modules do not share process state
- Cross-module workflows deferred to v1.1
- Example: A timesheet entry does NOT automatically trigger a work order. They are separate CASEs.

**Why:** Simplicity, clarity, testability, auditability.

**When this changes:** v1.1 will define linked workflows (onboarding triggers timesheet setup, work order completion triggers invoice, etc.)

---

## Reading Module Specifications

Each module has:

1. **Charter** (this document, quick overview)
2. **Governance Architecture** — statutory authority, domain model, enforcement rules
3. **Encoding Specification** — state machine, JSON schemas, validation rules, algorithms
4. **Process Flowchart** — visual state machine (in consolidated FLOWCHARTS_ALL_MODULES_v1.md)
5. **System Architecture** — three-rail diagram (in consolidated SYSTEM_ARCHITECTURES_ALL_MODULES_v1.md)

---

## Implementation Readiness

| Module | Status | Recommendation |
|--------|--------|---|
| **PRR** | Fully specified | Implement immediately (reference implementation) |
| **CLERK** | Fully specified | Implement after PRR (licensing is common need) |
| **FISCAL** | Fully specified | Implement 2nd or parallel with CLERK (financial controls) |
| **TIME** | Fully specified | Implement in parallel (straightforward, payroll ready) |
| **FIX** | Fully specified | Implement in parallel (simple SLA tracking) |
| **ONBOARD** | Fully specified | Implement in parallel (simplest module) |

---

*All modules follow the same governance and encoding patterns. Master PRR; other modules are specializations.*
