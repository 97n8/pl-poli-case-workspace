# What Is Out of Scope

This section clarifies what is **deliberately NOT** in this packet and why.

---

## UI/UX Design

**Not included:** Wireframes, mockups, interaction flows, styling, accessibility guidance.

**Why:** VAULT is a governance operating system, not a product. UI is a layered choice that depends on:
* Target users (staff vs. public)
* Organizational preference
* Regulatory guidance (e.g., accessibility standards)
* Brand and design system

VAULT specifies *what data must exist* and *what rules must enforce*.
You specify *how users interact with that data*.

**Example:** VAULT says "T10 deadline is enforced; if missed, FeesAllowed = false."
UI design says "Show a warning badge" or "Disable the invoice button" or "Log an alert to the supervisor."

---

## Database Schema

**Not included:** SQL schema, table definitions, column types, indexes, query optimization.

**Why:** VAULT specifies behavior contracts, not storage implementation.

As long as your storage satisfies:
* CASE entity contains required attributes
* LOCKED assets cannot be mutated
* Audit log is immutable and complete
* Timers compute correctly

Your schema is valid.

**Example:** You can implement CASE as:
* A single PostgreSQL row with JSONB documents
* A MongoDB document with nested collections
* An event store with immutable events plus computed state
* A graph database with linked entities

All are valid if they enforce the contracts.

---

## Retention Schedules by Statute

**Not included:** Detailed retention requirements for specific document types (e.g., "Board meeting minutes: 10 years per M.G.L. c. 39, §23A").

**Why:** Retention is municipal-specific, statute-specific, and changes frequently.

VAULT specifies *retention classes*:
* **KEEPER** — permanent
* **REFERENCE** — 7 years minimum
* **TRANSACTIONAL** — 1-2 years minimum

Each module assigns record types to classes. You (or municipal counsel) map those classes to your retention system.

**Example:** A PRR response letter is KEEPER (permanent retention). Your retention system may delete after 20 years per municipal policy, but VAULT does not enforce that.

---

## Configuration and Customization

**Not included:** Configuration files, feature flags, environment-specific settings, tenant customization.

**Why:** VAULT is a framework, not a product. Customization happens at deployment time, not in this packet.

VAULT is fixed; configuration is variable.

**Example:** VAULT says "T10 is 10 business days per M.G.L. c. 66, §10."
Configuration says "Our calendar observes July 4, Thanksgiving, and Winter Town Meeting."

---

## Local Policy and Bylaw

**Not included:** Municipal-specific procedures, local board policies, custom workflows, exception handling.

**Why:** VAULT is state-level governance architecture. Local policy layers on top without violating governance.

VAULT says: "If a PRR response takes > 25 days, you must obtain agreement or file a petition."
Local policy might say: "We always aim for 10 days, and we meet weekly to forecast completion."

Both are compatible. Local policy does not override VAULT.

---

## Integration Specifications

**Not included:** API contract definitions, webhook payloads, external system integration details, EDI formats.

**Why:** Integration depends on your technology choices and municipal infrastructure.

VAULT specifies *what must happen* (e.g., "At payment receipt, transition from PAYMENT_WAIT to DELIVERY").
You specify *how it happens* (e.g., "Stripe webhook triggers a state transition via REST API").

---

## Automation and Batch Processing

**Not included:** Automation recipes, scheduled jobs, batch correction procedures, mass redaction workflows.

**Why:** These are operational decisions that depend on scale, staffing, and tooling.

VAULT specifies *rules*; you specify *how to enforce them at scale*.

**Example:** VAULT says "Corrections must create a version chain and audit trail."
You decide: "We run a nightly batch job to apply validated corrections from a spreadsheet."

As long as each correction creates an audit entry and is linked to source, you're compliant.

---

## Training and Change Management

**Not included:** Training materials, change management plans, communication templates, rollout procedures.

**Why:** These are organizational, not technical.

Once VAULT is implemented, PublicLogic typically works with you on:
* Staff training (from Polumorphic or PublicInsight)
* Change communication (written by you for your organization)
* Rollout planning (depends on your readiness)

This packet supports those efforts but is not the effort itself.

---

## Edge Cases, Appeals, and Deferred Scope

**Not included:** Appeals workflow, request withdrawal, destruction schedules, re-opening closed cases.

**Why:** Version 1.0 focuses on core PRR lifecycle: intake → response → delivery → closure.

Edge cases (appeals, withdrawal, destruction) are deferred to v1.1 and handled manually in the interim.

**Specifically excluded from v1.0:**
* Appeal process (M.G.L. c. 66, §10(d) appeal to Supervisor)
* Withdrawal or reopening of closed cases
* Destruction procedures at end of retention period
* Exceptional extensions beyond T25 + granted extension

These will be handled as manual escalations. When volume justifies, they will be automated in v1.1.

---

## Pricing and Procurement

**Not included:** Pricing models, licensing terms, procurement process, SLA requirements.

**Why:** Those are commercial, not technical.

This packet is the technical foundation for commercial agreements, but is not itself commercial.

---

## Compliance and Audit Preparation

**Not included:** Audit checklists, compliance mapping to specific regulations, audit procedure documentation.

**Why:** Compliance preparation is done *after* implementation is locked.

This packet provides the *audit trail capability* (Audit_and_Evidence_Model.md).
You build *audit procedures* on top of that capability.

---

## What This Means for You

### Do Not Expect

* UI mockups
* Database schema
* Configuration guides
* Training materials
* Customization instructions
* Integration walkthroughs
* Deployment runbooks

### Do Expect

* Governance requirements
* Entity and attribute definitions
* Process flows and decision gates
* Enforcement rules and their consequences
* Record classification and immutability rules
* Audit trail specifications
* Timer computation and tolling rules
* Escalation procedures

---

## How to Think About "Out of Scope"

Everything in this packet answers: **"What is the system required to do?"**

Everything out of scope answers: **"How will you implement, configure, integrate, and operate this system?"**

The first is fixed. The second is flexible.

If you find yourself wanting to change something in this packet, stop and escalate. If you find yourself wanting to change something about implementation, you probably have the flexibility to do so.

---

*Out of Scope items become scope when: (1) multiple implementations ask for them, (2) they affect governance, or (3) they are blockers to compliance. Escalate if you hit that threshold.*
