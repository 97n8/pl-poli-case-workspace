# VAULT Core Canon: Cross-Module Interactions and Governance

**How Modules Relate • Single vs. Multi-Module Workflows**

---

## Core Principle

**v1.0: Modules are independent. One work unit = one CASE in one module.**

This is intentional. Independence simplifies implementation and governance.

---

## Module Independence (v1.0)

Each VAULT module operates as a self-contained system:

| Module | Work Unit | CASE Lifecycle |
|--------|-----------|---|
| **PRR** | One records request | Intake → Delivery → Closure |
| **CLERK** | One application | Intake → Approval/Denial → Closure |
| **FISCAL** | One invoice | Intake → Payment → Closure |
| **TIME** | One timesheet | Entry → Approval → Closure |
| **FIX** | One work order | Intake → Completion → Closure |
| **ONBOARD** | One employee | Intake → Completion → Closure |

**Key:** Modules do not exchange data or hand off CASEs between modules.

---

## Business Reality: People Are in Multiple Modules

An employee exists in multiple modules simultaneously:

```
Employee Jane Doe:
├── ONBOARD CASE (started Feb 1)
├── TIME CASE (timesheet for week of Feb 3)
├── FISCAL CASE (invoice she submitted on Feb 5)
└── FIX CASE (work request she submitted on Feb 10)
```

All four CASEs are independent. Each has its own lifecycle, deadlines, gates, and audit trail.

**This is correct.** Do not try to merge them or cross-reference them except at query level.

---

## Query-Level Correlation (v1.0 Support)

Systems may query across modules for reporting, but no process-level interaction:

**Example Queries:**

```
"Show me all open CASEs for employee Jane Doe"
  → Results: 1 ONBOARD, 1 TIME, 1 FIX (no FISCAL because closed)

"Show me all CASEs closed last week"
  → Results: 47 TIME, 12 FISCAL, 8 PRR, 2 CLERK

"Show me all CASEs by actor RAO-001"
  → Results: All PRR cases where RAO-001 took action
```

These queries are useful for reporting and auditing. They do NOT create dependencies between modules.

---

## What Cross-Module Workflows Are NOT Supported in v1.0

### Scenario 1: Onboarding Triggers Timesheet

**Scenario:** When ONBOARD CASE closes (employee ready to work), automatically create first TIME CASE.

**v1.0 Status:** NOT SUPPORTED.

**Why:** Creates temporal and dependency coupling between modules.

**Workaround:** Manual handoff. When onboarding completes, supervisor creates timesheet CASE.

**v1.1 Plans:** May automate this via explicit "workflow trigger" rules.

---

### Scenario 2: PRR Response Triggers Invoice

**Scenario:** When PRR response is delivered, automatically create FISCAL CASE for fees.

**v1.0 Status:** NOT SUPPORTED.

**Why:** Tightly couples PRR closure to FISCAL intake.

**Workaround:** Manual process. Finance team creates invoice CASE separately using fee information from PRR.

**v1.1 Plans:** May add "automated handoff" pattern (explicit governance).

---

### Scenario 3: Work Order (FIX) Triggers Timesheet (TIME)

**Scenario:** When work order is assigned, automatically create time entry records.

**v1.0 Status:** NOT SUPPORTED.

**Why:** Assumes TIME entries are only created via FIX; not true (staff can create timesheets independently).

**Workaround:** Separate workflows. FIX CASE and TIME CASE are created independently.

**v1.1 Plans:** May support "optional linking" (not triggering).

---

## Why Independence in v1.0?

### Simplicity

Each module is simpler to implement if it does not depend on others.

### Clarity

Governance is clearer when one CASE = one module. No ambiguity about "who owns this state?"

### Auditability

Each CASE's audit trail is self-contained. No "did module A cause module B to do X?"

### Testability

Testing modules independently is easier. No cross-module test matrix.

### Risk Mitigation

If one module has a bug, it does not cascade to others.

---

## Business Process During v1.0

**Current (Independent Modules):**

```
Employee Onboarding:
  1. HR creates ONBOARD CASE
  2. Onboarding completes
  3. ONBOARD CASE closes
  4. [Manual: Finance creates FISCAL CASE for new employee setup costs, if any]
  5. [Manual: Payroll creates TIME CASE for first timesheet]

This is okay in v1.0. It's a bit manual but clear and safe.
```

---

## Future (v1.1+): Linked Workflows

**Planned for v1.1:**

```
Explicit handoff rules:
  - When ONBOARD closes, notify (not auto-create) TIME supervisor
  - When PRR closes with fees, create FISCAL record (but require approval)
  - When FIX completes, optionally link to TIME for labor tracking

This allows coupling without tight temporal dependencies.
```

---

## Cross-Module Data Sharing

### Data That Can Be Shared (v1.0)

* **Actor identities** — Same RAO works in PRR and CLERK
* **Configuration** — Same holiday calendar used in all modules
* **Query results** — "Show all CASEs for actor X" works across modules

### Data That Cannot Be Shared (v1.0)

* **CASEs** — One CASE per module per work unit
* **Process state** — One module's state machine is independent
* **Timers and gates** — Specific to each module
* **Assets and audit trails** — Contained within one module

### Data That MUST NOT Be Shared

* **Audit log entries** — Each module's audit is separate
* **LOCKED assets** — Cannot be re-locked in different module
* **Retention class decisions** — Made per module per governance

---

## Governance Rule: Module Boundary

**Rule:** A CASE cannot span multiple modules.

**Enforcement:**
- CaseType is set at creation (ONBOARD, TIME, FISCAL, PRR, CLERK, FIX)
- CaseType cannot change
- All assets within a CASE belong to the case's module

**Why:** Clear ownership and governance.

---

## Q&A

**Q: Can I create a CASE that is "part ONBOARD, part TIME"?**
A: No. If it involves onboarding, it's ONBOARD CASE. If it involves timesheets, it's TIME CASE. Create separate CASEs.

**Q: Can ONBOARD CASE trigger TIME CASE creation automatically?**
A: Not in v1.0. This is a v1.1 feature. For now, manual handoff.

**Q: Can I query across modules?**
A: Yes. Query-level correlation is fine. "Show me all CASEs by Jane Doe" is valid. Just do not try to merge them into one CASE.

**Q: What if an employee is fired mid-onboarding?**
A: ONBOARDING CASE closes as "Cancelled." TIME CASE (if created) closes separately. No cross-module reversal.

**Q: Can I use one CASE for both work order and timesheet?**
A: No. FIX CASE for the work. TIME CASE for the timesheet. Link them at query level if needed, but separate CASEs.

---

## Migration Plan to Multi-Module Workflows (v1.1)

When v1.1 adds cross-module support:

1. **Governance amendment** — Define explicit "workflow triggers"
2. **Specification updates** — Detail trigger conditions and handoff protocols
3. **Implementation updates** — Polymorphic adds trigger evaluation
4. **Testing** — New test cases for cross-module flows
5. **Deployment** — Clients update configuration to enable triggers

Until v1.1: Use manual handoffs. They are safe and clear.

---

## Reporting Across Modules

Even in v1.0, you can report across modules:

**Report: Employee Lifecycle** (query example)

```
Employee: Jane Doe

ONBOARD:
  CaseID: onboard-001
  Status: CLOSED
  Duration: 3 weeks

TIME:
  CaseID: time-2026-w06-001
  Status: OPEN
  Hours: 38/40

FISCAL:
  CaseID: fiscal-2026-0145
  Status: CLOSED (expense reimbursement)
  Amount: $250

FIX:
  (no cases)

This report ties CASEs together by actor (Jane Doe) but does not merge them.
```

**This is good reporting. Do it.**

---

## Governance: Cross-Module Boundaries Are Intentional

Do not circumvent module independence by:
* Storing data from one module in another module's CASE
* Creating "super-CASEs" that span modules
* Having one module depend on another module's CASE state
* Allowing process flow to jump between modules

If you need cross-module workflow:
1. **v1.0:** Use manual handoff
2. **v1.1:** Escalate to PublicLogic for governed cross-module support

---

*Module independence is intentional and protective. Maintain it through v1.0. Evolution to linked workflows happens via formal governance amendment in v1.1.*
