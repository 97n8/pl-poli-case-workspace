# VAULT Core Canon: Authority and Scope

**Why VAULT Exists • What It Does and Does Not Do**

---

## Authority

VAULT is published and enforced by **PublicLogic LLC**.

PublicLogic holds governance authority over:

* Framework definition and architecture
* Statutory interpretation and enforcement rules
* Module specifications and process flows
* Change control and version management
* Compliance validation and audit requirements

**PublicLogic does not:**
* Implement VAULT (that is Polymorphic's job)
* Make municipal policy (that is the client's job)
* Design user interfaces (that is collaborative)
* Configure client-specific settings (that is the client's job)

---

## Core Purpose

VAULT solves one problem: **Risk transfer from individuals to structure.**

In traditional municipal systems, risk concentrates on individuals:
* "Did the RAO remember the deadline?"
* "Did the invoice processor catch the missing PO?"
* "Did anyone document this decision?"

If the person leaves, the system breaks.

VAULT moves risk into structural enforcement:
* Deadlines are computed and enforced automatically
* Decisions are recorded immutably
* Processes are auditable and defensible
* The system continues when staff changes

**The goal is not efficiency. The goal is institutional survivability.**

---

## What VAULT Does

### 1. Enforces Statutory Deadlines

VAULT computes and enforces statutory deadlines as hard constraints, not reminders.

Example (VAULTPRR):
* M.G.L. c. 66, §10 requires initial response within 10 business days
* VAULT computes the exact due date
* If deadline is missed, consequences execute automatically (fee waiver)
* No human can bypass this rule

**This is enforcement, not tracking.**

### 2. Records All Decisions Immutably

When a decision is made, it is recorded and locked.

Example (VAULTPRR):
* RAO decides whether records are exempt
* Decision is recorded as an immutable asset
* Decision cannot be edited (only versioned if changed later)
* Audit log shows who decided, when, and why

**This creates defensible evidence.**

### 3. Prevents Process Divergence

Processes follow defined paths. Shortcuts are not permitted.

Example (VAULT FISCAL):
* Invoice must go through: Intake → Validation → Approval → Posting → Payment
* Cannot skip approval
* Cannot back up once posted
* Each step creates artifacts and audit entries

**This prevents workarounds and heroics.**

### 4. Ensures Auditability

Everything is recorded. The system can answer "What happened?" and "Who did it?" for any work unit.

Example (VAULT CLERK):
* License application shows every interaction
* Every decision, approval, denial has timestamp and actor
* If auditor asks "Why was this approved?", answer exists in CASE

**This satisfies compliance and legal discovery.**

### 5. Enables Institutional Transfer

When staff leave, their work is already recorded and documented.

Example (VAULT ONBOARD):
* New hire's onboarding is fully documented in CASE
* Replacement hire can see every step that was completed
* No institutional knowledge lost

**This is survival.**

---

## What VAULT Does NOT Do

### Not About Speed

VAULT may slow down individual processes because it requires recording and enforcement.

If your goal is "process things as fast as possible," VAULT is not for you.

**VAULT's goal:** "Process things in a way that survives scrutiny."

---

### Not About Flexibility

VAULT defines specific process flows. You cannot rearrange steps or skip gates.

If you need flexibility to handle exceptions, you request formal amendment through governance gap protocol.

**VAULT's goal:** "Be predictable and auditable."

---

### Not About Optimization

VAULT is not designed for performance optimization or cost reduction.

You can optimize *how* you implement VAULT (database choice, caching, etc.), but not the *what*.

**VAULT's goal:** "Be safe, not efficient."

---

### Not About Convenience

VAULT creates friction. Decisions must be recorded. Deadlines must be met. Assets must be locked.

This friction is intentional. It prevents shortcuts that transfer risk to individuals.

**VAULT's goal:** "Make risk visible and structural."

---

## Scope: What VAULT Covers

VAULT addresses **regulated municipal work** — functions where:

1. **Statute defines the process** (e.g., Public Records Law, Licensing)
2. **Deadlines are mandatory** (e.g., response windows, appeal periods)
3. **Decisions must be defensible** (e.g., withholding reasons, approval authority)
4. **Risk is high** (e.g., legal challenges, staff turnover, audit failure)

**Modules in VAULT v1.0:**

| Module | Authority | Why |
|--------|-----------|-----|
| **PRR** | M.G.L. c. 66, §10; 950 CMR 32.00 | Public records are statutory; deadlines are hard |
| **CLERK** | M.G.L. c. 101 (licensing); various | License decisions must be defensible |
| **FISCAL** | M.G.L. c. 41, §23 (AP); Budget Law | Procurement and payment must be auditable |
| **TIME** | M.G.L. c. 149 (labor); Payroll | Compensation must be auditable |
| **FIX** | Municipal operations | Work orders provide evidence trail |
| **ONBOARD** | HR best practices | Onboarding is institutional transfer risk |

---

## Scope: What VAULT Does NOT Cover

### Casual / Internal Processes

VAULT is not for:
* Informal team communication
* Meeting notes that are not formal decisions
* Brainstorming or exploratory work
* Day-to-day operational decisions with no legal consequence

**Example:** Team lunch planning does not go in VAULT.

### High-Volume Transactions

VAULT is not for:
* Individual email messages
* Routine maintenance work items
* Every status update

VAULT is for **work units that have legal or administrative significance.**

**Example:** Processing 1000 parking ticket payments would create 1000 CASEs. That is correct — each has audit and closure significance.

---

## Design Principles

### 1. Structure is Care

Proper structure protects people. Lack of structure creates risk.

When you enforce a deadline in VAULT, you are protecting the staff member from missing it.

When you record a decision immutably, you are protecting the decision-maker from later being accused of changing it.

**Structure is not punitive. It is protective.**

### 2. Recordable, Transferable, Defensible

If something is not recordable in VAULT, it is not safe work.

If something is recorded in VAULT, it is transferable (staff can change).

If something is transferred and recorded, it is defensible (auditable and explainable).

### 3. Statutory Authority First

VAULT modules start with statute, not with "how we currently do it."

If statute requires something, VAULT enforces it. You do not have the option to skip it.

If you currently do something that statute does not require, ask whether VAULT needs to enforce it. Usually not.

### 4. Enforcement Before Discretion

Rules execute before human judgment applies.

Example: T10 deadline is checked before fees are calculated. If deadline is missed, fees are waived automatically. There is no opportunity for someone to say "Let's charge fees anyway."

This is protection.

### 5. No Workarounds

VAULT processes are not negotiable. You cannot create a special path for "urgent" cases or "difficult" scenarios.

If you need a special path, that becomes a new module version with formal governance.

This protects you from scope creep and from creating maintenance burden.

---

## How VAULT Relates to Your Existing Processes

### VAULT Does Not Replace Everything

Your municipality has processes that predate VAULT. VAULT does not necessarily replace all of them.

VAULT enforces statutory requirements and high-risk functions.

Lower-risk functions can stay as-is.

### VAULT Is Additive

VAULT layers on top of existing infrastructure.

Your email system, phone system, records storage can stay as-is.

VAULT defines how decisions are recorded and enforced.

### VAULT Can Coexist

VAULT and non-VAULT work can operate in parallel.

Some municipalities run VAULT PRR and non-VAULT internal memo tracking.

Both are fine. Separation is clear.

---

## Making Changes to VAULT

### What Requires Amendment

* Changes to deadline enforcement
* Changes to gate logic
* Changes to record classification
* Changes to closure criteria
* Changes to audit requirements

**Process:** Escalate to PublicLogic via governance gap protocol.

### What Does Not Require Amendment

* Configuration (e.g., which holidays are non-business days)
* Integration (e.g., how to connect to payroll system)
* Implementation (e.g., database choice, API design)
* Organizational policy (e.g., "our target is 5-day response")

**Process:** Decide and implement.

---

## Boundaries with Partners

### PublicLogic (Framework)

PublicLogic owns:
* VAULT architecture
* Governance rules
* Statutory interpretation
* Process flows
* Change control

### Polymorphic (Runtime)

Polymorphic owns:
* State machine implementation
* Data schema design
* Timer algorithms
* Integration architecture
* Deployment and operations

### Client (Deployment)

Client owns:
* Configuration (holidays, cost codes, approval chain)
* Integration (connections to existing systems)
* Policy (response time targets, retention schedules)
* Training and change management

---

*Authority and Scope define what VAULT is and what it is not. If you disagree with this scope, escalate before implementing.*
