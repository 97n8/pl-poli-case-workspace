# VAULT Glossary

---

## Core Entities

**CASE** — The root container for a single work unit in VAULT. Every meaningful piece of work lives inside exactly one CASE. CASEs are immutable once closed.

**Asset** — Any document, decision, or artifact within a CASE. Assets have types (IntakeForm, Letter, Decision, etc.), retention classes, and lock status.

**LOCKED** — Immutable status. A LOCKED asset cannot be edited. Corrections require creating a new version.

**KEEPER** — Permanent retention class. KEEPER records are retained indefinitely.

**REFERENCE** — 7-year minimum retention class. REFERENCE records are retained for 7 years from CASE closure.

**TRANSACTIONAL** — 1-2 year minimum retention class. TRANSACTIONAL records are retained 1-2 years from CASE closure.

---

## Deadlines and Timers

**Deadline** — A statutory or business requirement with a specific due date and consequence.

**Timer** — The mechanism that tracks a deadline and enforces consequences.

**T10** — Initial response deadline (10 business days from effective receipt). Miss triggers fee waiver.

**T20** — Petition window (20 business days from effective receipt). Close after this date ends right to petition.

**T25** — Municipal production deadline (25 business days from effective receipt). Forecast past this requires extension.

**T90** — Appeal window (90 calendar days from CASE closure). Close after this date ends right to appeal.

**EffectiveReceiptDate** — The date from which all timers start. If request received on weekend/holiday, shifts to next business day.

**Business Day** — Weekday (Monday–Friday) that is not a state legal holiday or municipal closure.

**Tolling** — Pausing the deadline clock. Used when clarification needed, requester requests hold, etc. When paused, days are added back to deadline.

**Enforcement** — The automatic execution of consequences when deadline is missed. No discretion.

---

## Process and States

**State** — Current position of a CASE in the process (e.g., INTAKE, ASSESSMENT, GATHERING, DELIVERY, CLOSED).

**Transition** — Movement from one state to another. Transitions can be blocked by gates.

**Gate** — A decision point that must be passed before transition. Examples: T25 forecast, exemption review, payment received.

**Blocker** — A condition preventing transition. All blockers must be resolved before CASE can transition.

**Step** — An action within a process. Examples: FormStep (fill form), ReviewStep (review records), ManualStep (search).

---

## Governance

**Authority** — Holder of responsibility for framework definition. PublicLogic holds VAULT authority.

**Governance Gap** — An ambiguity in specification that encoding team encounters. Must be escalated to PublicLogic for clarification.

**Governance Gap Protocol** — Process for escalating ambiguities: Stop → Document → Escalate → Wait → Resume.

**Specification** — Detailed document defining how something must work. Specifications come in tiers: Governance (why), Encoding (how), Derivative (implementation).

---

## Records and Immutability

**Immutable** — Cannot be changed after creation. LOCKED assets are immutable. Audit log is immutable.

**Version Chain** — History of all versions of an asset, linked together. When LOCKED asset is corrected, new version created and linked.

**Retention** — How long a record is kept. Determined by retention class (KEEPER, REFERENCE, TRANSACTIONAL).

**Destruction** — Final deletion of records at end of retention period. Must be logged.

**Record Classification** — Assignment of asset to retention class (KEEPER, REFERENCE, TRANSACTIONAL).

---

## Audit and Compliance

**Audit Log** — Immutable chronological record of every action in a CASE. Includes timestamp, actor, action, resource, reason.

**Event** — Single entry in audit log. Events cannot be edited or deleted.

**Actor** — Identity of person or system that performed an action. Must be logged. No anonymous actions.

**Evidence Trail** — Complete record of what happened, who did it, when, and why. Audit log is the primary evidence trail.

**Compliance** — Conformance to statute and governance rules. VAULT is designed for compliance.

---

## Modules

**Module** — Specialized instantiation of VAULT CASE for specific work type. Examples: PRR (public records), CLERK (licensing), FISCAL (invoices), TIME (timesheets), FIX (work orders), ONBOARD (employee onboarding).

**Core Canon** — Universal CASE architecture and rules shared by all modules.

**Module-Specific Rules** — Rules unique to a particular module (e.g., exemption analysis in PRR, 3-way match in FISCAL).

---

## Statutory and Legal

**Exemption** — Legal ground to withhold information from public records request. Examples: trade secrets, personnel files, deliberative materials.

**Withholding** — Refusal to disclose records. Must be accompanied by written explanation citing exemption.

**Segregability** — Requirement to disclose non-exempt portions even if parts are exempt.

**M.G.L. c. 66, §10** — Massachusetts Public Records Law. Authority for VAULTPRR.

**950 CMR 32.00** — Supervisor of Records Regulations. Authority for VAULTPRR appeals.

**Supervisor of Records** — State official who oversees public records and hears appeals.

**Records Access Officer** — Municipal official designated to handle public records requests. Role in VAULTPRR.

---

## Technical Terms

**Schema** — Structure of data. VAULT defines entity schemas (CASE, Asset, LogEntry) but not storage schema.

**Encoding** — Concrete implementation of governance architecture in code/configuration. Encoding Specification details how this is done.

**Implementation** — Bringing VAULT to life in a specific technology stack.

**Deployment** — Installing VAULT in a specific municipality.

**Configuration** — Municipality-specific settings (holidays, cost codes, approval chains) that do not change governance.

---

## Roles (Module Examples)

**Records Access Officer (RAO)** — VAULTPRR role. Designated municipal official who handles PRRs.

**Supervisor of Records** — State official. Oversees PRRs and hears appeals.

**Approver** — VAULT FISCAL role. Person authorized to approve invoices or payments.

**Requester** — VAULTPRR role. Person requesting public records.

**Applicant** — VAULT CLERK role. Person applying for license/permit.

---

## Q&A Examples

**Q: What is CASE?**
A: Root container for one work unit. PRR request = one CASE. Invoice = one CASE. License application = one CASE.

**Q: What does LOCKED mean?**
A: Cannot be edited. Corrections require new version.

**Q: How long do I keep records?**
A: Depends on retention class. KEEPER = forever. REFERENCE = 7 years. TRANSACTIONAL = 1-2 years.

**Q: What is T25?**
A: Deadline for production of records in PRR. 25 business days from effective receipt.

**Q: Can I skip audit logging for performance?**
A: No. Audit log is mandatory and immutable.

**Q: What is a Governance Gap?**
A: Ambiguity in spec. Escalate to PublicLogic instead of guessing.

---

*This glossary defines standard VAULT vocabulary. Use these definitions consistently.*
