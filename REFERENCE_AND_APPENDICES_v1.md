# VAULT Reference & Appendices v1.0

---

## Part 1: Glossary

### Core Entities

**CASE** — The root container for a single work unit in VAULT. Every meaningful piece of work lives inside exactly one CASE. CASEs are immutable once closed.

**Asset** — Any document, decision, or artifact within a CASE. Assets have types (IntakeForm, Letter, Decision, etc.), retention classes, and lock status.

**LOCKED** — Immutable status. A LOCKED asset cannot be edited. Corrections require creating a new version.

**KEEPER** — Permanent retention class. KEEPER records are retained indefinitely.

**REFERENCE** — 7-year minimum retention class. REFERENCE records are retained for 7 years from CASE closure.

**TRANSACTIONAL** — 1-2 year minimum retention class. TRANSACTIONAL records are retained 1-2 years from CASE closure.

### Deadlines and Timers

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

### Process and States

**State** — Current position of a CASE in the process (e.g., INTAKE, ASSESSMENT, GATHERING, DELIVERY, CLOSED).

**Transition** — Movement from one state to another. Transitions can be blocked by gates.

**Gate** — A decision point that must be passed before transition. Examples: T25 forecast, exemption review, payment received.

**Blocker** — A condition preventing transition. All blockers must be resolved before CASE can transition.

**Step** — An action within a process. Examples: FormStep (fill form), ReviewStep (review records), ManualStep (search).

### Governance

**Authority** — Holder of responsibility for framework definition. PublicLogic holds VAULT authority.

**Governance Gap** — An ambiguity in specification that encoding team encounters. Must be escalated to PublicLogic for clarification.

**Governance Gap Protocol** — Process for escalating ambiguities: Stop → Document → Escalate → Wait → Resume.

**Specification** — Detailed document defining how something must work. Specifications come in tiers: Governance (why), Encoding (how), Derivative (implementation).

### Records and Immutability

**Immutable** — Cannot be changed after creation. LOCKED assets are immutable. Audit log is immutable.

**Version Chain** — History of all versions of an asset, linked together. When LOCKED asset is corrected, new version created and linked.

**Retention** — How long a record is kept. Determined by retention class (KEEPER, REFERENCE, TRANSACTIONAL).

**Destruction** — Final deletion of records at end of retention period. Must be logged.

**Record Classification** — Assignment of asset to retention class (KEEPER, REFERENCE, TRANSACTIONAL).

### Audit and Compliance

**Audit Log** — Immutable chronological record of every action in a CASE. Includes timestamp, actor, action, resource, reason.

**Event** — Single entry in audit log. Events cannot be edited or deleted.

**Actor** — Identity of person or system that performed an action. Must be logged. No anonymous actions.

**Evidence Trail** — Complete record of what happened, who did it, when, and why. Audit log is the primary evidence trail.

**Compliance** — Conformance to statute and governance rules. VAULT is designed for compliance.

### Modules

**Module** — Specialized instantiation of VAULT CASE for specific work type. Examples: PRR (public records), CLERK (licensing), FISCAL (invoices), TIME (timesheets), FIX (work orders), ONBOARD (employee onboarding).

**Core Canon** — Universal CASE architecture and rules shared by all modules.

**Module-Specific Rules** — Rules unique to a particular module (e.g., exemption analysis in PRR, 3-way match in FISCAL).

### Statutory and Legal

**Exemption** — Legal ground to withhold information from public records request. Examples: trade secrets, personnel files, deliberative materials.

**Withholding** — Refusal to disclose records. Must be accompanied by written explanation citing exemption.

**Segregability** — Requirement to disclose non-exempt portions even if parts are exempt.

**M.G.L. c. 66, §10** — Massachusetts Public Records Law. Authority for VAULTPRR.

**950 CMR 32.00** — Supervisor of Records Regulations. Authority for VAULTPRR appeals.

**Supervisor of Records** — State official who oversees public records and hears appeals.

**Records Access Officer** — Municipal official designated to handle public records requests. Role in VAULTPRR.

### Technical Terms

**Schema** — Structure of data. VAULT defines entity schemas (CASE, Asset, LogEntry) but not storage schema.

**Encoding** — Concrete implementation of governance architecture in code/configuration. Encoding Specification details how this is done.

**Implementation** — Bringing VAULT to life in a specific technology stack.

**Deployment** — Installing VAULT in a specific municipality.

**Configuration** — Municipality-specific settings (holidays, cost codes, approval chains) that do not change governance.

### Roles (Module Examples)

**Records Access Officer (RAO)** — VAULTPRR role. Designated municipal official who handles PRRs.

**Supervisor of Records** — State official. Oversees PRRs and hears appeals.

**Approver** — VAULT FISCAL role. Person authorized to approve invoices or payments.

**Requester** — VAULTPRR role. Person requesting public records.

**Applicant** — VAULT CLERK role. Person applying for license/permit.

### Quick Q&A

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

## Part 2: Version History and Status

### Current Version

**VAULT Master Engineering Packet v1.0**  
**Released:** January 2026  
**Authority:** PublicLogic LLC  
**Status:** Authoritative Governance

### What Is Included in v1.0

**VAULT Core Canon:**
* CASE Workspace architecture (universal, all modules)
* Authority and Scope (why VAULT exists)
* Record Classes and Immutability (KEEPER, REFERENCE, TRANSACTIONAL)
* Timer and Deadline Model (T10, T20, T25, T90 enforcement)
* Audit and Evidence Model (immutable logging)

**Modules (All Included and Production-Ready):**
* VAULTPRR — Public Records Requests (reference implementation)
* VAULT CLERK — Licensing
* VAULT FISCAL — Accounts Payable and Procurement
* VAULT TIME — Timesheets and Compensation
* VAULT FIX — Work Orders
* VAULT ONBOARD — Employee Onboarding

**Implementation Guidance:**
* What Engineers May Choose (technical flexibility)
* What Is Non-Negotiable (governance constraints)
* Testing, Error Handling, Trade-off Analysis, Escalation SLA

### Scope Limitations (v1.0)

The following are **deferred to v1.1** and handled manually:

* **VAULTPRR Appeals** — Appeal process after closure. Notifications sent; cases do not reopen in v1.0.
* **Request Withdrawal** — Ability to withdraw PRR after intake. Treated as edge case requiring escalation.
* **Destruction Workflows** — Automated destruction at end of retention period. Manual deletion process used instead.
* **Reopening Cases** — Once closed, cases cannot reopen (v1.0 design). Appeals and corrections treated as new cases or escalations.
* **Cross-Module Workflows** — v1.0 modules are independent. v1.1 will define linked workflows.

### Known Constraints

1. **Single-Jurisdiction Focus** — Spec assumes single municipality. Multi-jurisdiction or regional deployments require configuration/amendment.

2. **No Concurrent Editing** — Multiple simultaneous edits to same CASE may cause conflicts. Recommend serial editing within modules.

3. **Bulk Operations** — No bulk processing workflows for multiple CASEs simultaneously. Each CASE processed individually or through batch jobs with audit trail.

4. **Notification Limits** — No built-in notification engine. Implementation must layer on standard SMTP, Slack, etc.

### How This Version Will Evolve

**v1.1 (Expected Q2 2026):**
* Appeals workflow — Reopening closed PRR cases to handle Supervisor appeals
* Withdrawal protocol — Handling request withdrawal mid-process
* Destruction automation — Scheduled destruction at end of retention period
* Bulk operations — Batch editing and mass CASE operations
* Cross-module workflows — Linked workflows between modules
* Additional modules — (TBD based on demand)

### Amendment Process

Changes to VAULT are versioned and documented.

**Change Request:** Escalate via governance gap protocol (info@publiclogic.com)

**Amendment Process:**
1. PublicLogic evaluates request
2. Changes are documented with rationale
3. Version number incremented
4. All active deployments notified
5. Amendments are backward-compatible where possible

### Breaking Changes

**None in v1.0 release.**

v1.0 is the first authoritative release. Future versions will note any breaking changes clearly.

### Known Issues and Workarounds

**Issue 1: Deadline Computation Edge Case**
* **Scenario:** If a request is received at 23:59:59 on Friday, is it "Friday" or "Monday"?
* **v1.0 Answer:** Business day is calendar day (00:00 to 23:59). Receipt on Friday is Friday. If you want strict time-of-day logic, escalate.
* **Workaround:** Configure municipal business hours (e.g., 9 AM to 5 PM). Receipt after hours shifts to next business day.

**Issue 2: Holiday Calendar Management**
* **Scenario:** Multiple municipalities may have different holidays.
* **v1.0 Answer:** Each municipality configures its own holiday calendar.
* **Workaround:** PublicLogic provides Massachusetts state holiday calendar. Municipalities add custom closures.

**Issue 3: Tolling Communication**
* **Scenario:** When clock is tolled (clarification requested), how does municipality notify requester? Is email automatic?
* **v1.0 Answer:** Spec defines tolling logic. Notification is outside scope (v1.0). Implementation must layer on notification engine.
* **Workaround:** Use standard email system. Create "Send Clarification Request" step that generates email and logs it as asset in CASE.

### Legal and Regulatory Status

**Compliance:** VAULT Master Engineering Packet v1.0 is designed to comply with:
* M.G.L. c. 66, §10 — Massachusetts Public Records Law
* M.G.L. c. 4, §7(26) — Exemptions
* 950 CMR 32.00 — Supervisor of Records Regulations
* M.G.L. c. 101 — Licensing (general; specific license types may have additional requirements)
* M.G.L. c. 41, §23 — Municipal Finance
* M.G.L. c. 149 — Labor and Fair Employment Practices

**Not Legal Advice:** This packet is a technical specification, not legal guidance. Municipal counsel should review governance requirements before implementation.

### Support and Maintenance

**PublicLogic Support:**
* Governance clarifications — Via governance gap protocol (info@publiclogic.com)
* Specification amendments — Via PublicLogic amendment process
* Compliance validation — Available upon request

**Implementation Partner Support:**
* Implementation questions — Via technical leads
* Integration issues — Via engineering team
* Operational support — Per service agreement

**Client Support:**
* Configuration questions — Via implementation team
* Operational procedures — Via PublicLogic onboarding
* Training — Via PublicLogic or internal team

---

*This glossary and version history are authoritative. Reference these when implementing or deploying VAULT.*
