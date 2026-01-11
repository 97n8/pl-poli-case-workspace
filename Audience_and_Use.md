# Audience and Use

## Primary Audience

**Polymorphic Engineering (and similar runtime partners)**

This packet is written for engineers, architects, and technical decision-makers who are responsible for:

* State machine implementation
* Data schema design and validation
* Timer computation and enforcement
* Integration with external systems
* Error handling and recovery
* Audit trail capture and maintenance

**You are the decision-maker on:**
* Storage architecture (relational, document, event-sourced)
* Integration patterns (APIs, webhooks, queues)
* Error recovery and retry logic
* Performance and scalability
* Deployment and operations

**You are NOT responsible for:**
* Statutory interpretation (that's governance)
* Business rule changes (those go through PublicLogic)
* End-user experience design (layered on top)
* Municipal policy decisions (client responsibility)

---

## Secondary Audiences

### Municipal Staff (Derivative)

Once VAULT is live, municipal staff use simplified documentation derived from this packet:

* Standard Operating Procedures (SOP)
* Training materials
* Quick reference guides
* Help text embedded in UI

**These are NOT in this packet.** They are built *after* engineering is locked.

### PublicLogic Internal (Expanded)

PublicLogic uses this packet as the foundation for:

* Client implementation guidance
* Change control and amendments
* Audit and compliance validation
* Sales and partnership documentation

### Legal and Governance (Reference)

Municipal counsel, auditors, and oversight bodies can reference this packet to understand:

* Statutory enforcement architecture
* Decision gate rationale
* Record retention and immutability rules
* Audit trail capabilities

---

## How This Packet Is Used

### At Implementation Time

Engineers use this packet to:

* Understand requirements before writing code
* Identify decision gates and their consequences
* Design data models that satisfy immutability rules
* Implement state machines without interpretation
* Validate implementation against specification

**Example workflow:**

1. Engineer reads "Timer_and_Deadline_Model.md"
2. Engineer identifies T10, T20, T25, T90 requirements
3. Engineer designs timer computation logic
4. Engineer implements state machine transitions guarded by timers
5. Engineer validates against Encoding Specification
6. Engineer escalates ambiguity if any arises

### At Deployment Time

Operations teams use this packet to:

* Understand what each module does
* Validate configuration against governance rules
* Set up audit logging and retention policies
* Identify escalation procedures

### At Audit Time

Auditors and compliance teams use this packet to:

* Verify CASE structure and content
* Validate timer enforcement
* Check immutability of locked assets
* Confirm audit trail completeness

---

## What You Should NOT Do

**Do not:**

* Interpret governance as "suggestions" or "best practices"
* Optimize around enforcement rules (e.g., skip T10 check to "save processing")
* Create special cases or exceptions to CASE structure
* Allow mutation of LOCKED assets
* Disable audit logging "for performance"
* Skip timer computation "if deadline appears far off"

**These are structural requirements.** They protect the municipality, its staff, and the system itself.

---

## What You SHOULD Do

**Do:**

* Read the VAULT Core Canon before coding
* Implement exactly as specified
* Escalate ambiguity immediately (do not guess)
* Test timer computation against real-world dates
* Validate CASE closure conditions
* Document your design decisions against this packet

---

## Governance vs. Technical

### Governance (This Packet)

* Why timers exist
* What constitutes enforcement
* What records must be immutable
* What facts must be audited
* What decisions are irreversible

### Technical (Your Responsibility)

* How to compute timers efficiently
* Whether to use SQL, NoSQL, or event sourcing
* How to scale audit logging
* How to optimize storage
* How to handle failure modes

**If you are asking:** "Can we skip the timer check?"
**Answer:** No. That is governance, not technical.

**If you are asking:** "Should we use PostgreSQL or DynamoDB?"
**Answer:** Either, as long as timers are enforced exactly as specified.

---

## Getting Help

### Questions About Implementation

* Scope: Does this state machine handle appeals?
* Design: Should timer computation happen at intake or at the next step?
* Integration: How should we connect to external records systems?

**Send to:** Polymorphic engineering leadership
**Resolution:** Internal engineering decision; may inform PublicLogic of constraints

### Questions About Governance

* Scope: Does M.G.L. c. 66, §10 allow partial disclosure if search fails?
* Requirements: Can we defer closure validation to a manual weekly review?
* Authority: Should appeals be handled by this module or escalated?

**Send to:** info@publiclogic.com
**Resolution:** PublicLogic clarifies governance; may amend specification

---

## Version and Change Control

**Current Version:** VAULT Master Engineering Packet v1.0 (January 2026)

**Changes to VAULT Core Canon:**
* Require PublicLogic approval
* Are versioned and documented
* Are communicated to all active implementations
* May require re-validation of running systems

**Changes to Module Specifications:**
* May be module-specific or core-level
* Follow same change control
* Are tracked in CHANGE_LOG.md

---

## Escalation Path

If you hit ambiguity:

```
Engineer identifies ambiguity
    ↓
Document: What is ambiguous? Why does it matter? What decision is needed?
    ↓
Send to: Polymorphic Technical Lead
    ↓
Polymorphic escalates to: info@publiclogic.com
    ↓
PublicLogic responds with: Clarification or Specification Amendment
    ↓
Resume implementation with clarification in hand
```

**Target turnaround:** 2-3 business days

---

*This document defines the intended use of VAULT Master Engineering Packet. If you are reading this and your use case differs, ask before proceeding.*
