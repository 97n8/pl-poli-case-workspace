# VAULT Escalation SLA

**Service Level Agreements for Governance Support**

---

## Overview

When you encounter a governance gap or need clarification, escalate to PublicLogic.

This document defines response times and severity tiers.

---

## Escalation Severity Tiers

### Severity 1: Critical Blocker (STOP ALL WORK)

**Definition:** Implementation cannot proceed safely without clarification. Deployment is halted.

**Examples:**
* "Spec does not define what to do if PRR is withdrawn mid-process"
* "Encoding introduces undocumented risk or dependency"
* "Deadline enforcement would create legal exposure if unclear"
* "Multi-module interaction is ambiguous"

**Response Time:** < 4 hours (same business day if during work hours)

**Resolution Time:** < 24 hours

**Escalation Method:** 
1. Document the blocker clearly (1-2 paragraphs)
2. State why work is halted
3. Email to info@publiclogic.com with subject: **[CRITICAL]** [Issue Title]
4. Page on-call if urgent (critical@publiclogic.com)

**Examples of Critical Response:**
```
Subject: [CRITICAL] PRR Withdrawal Protocol

Scenario: Requester submits PRR on Jan 15. On Jan 20, after search is complete, 
requester calls and says "never mind, withdraw the request."

Questions: 
1. Can PRRs be withdrawn after intake?
2. What happens to gathered records? Destroy?
3. Do we send confirmation letter?

Impact: Cannot close PRR case without guidance. Blocking 3 municipalities.

Please clarify by EOD today if possible.
```

---

### Severity 2: High Priority (SCHEDULE SOON)

**Definition:** Implementation can continue with workaround, but needs clarification before go-live.

**Examples:**
* "Spec doesn't fully define appeals process"
* "Module interaction rules are unclear"
* "Statutory interpretation question"
* "Error recovery procedure not in spec"

**Response Time:** Next business day

**Resolution Time:** < 3 business days

**Escalation Method:**
1. Document the question clearly (2-3 paragraphs)
2. State what workaround you're using temporarily
3. State go-live deadline if relevant
4. Email to info@publiclogic.com with subject: **[HIGH]** [Issue Title]

**Examples of High Response:**
```
Subject: [HIGH] CLERK Module - License Type Configuration

Question: Spec defines CASE structure for licensing but doesn't define 
how to configure license-type-specific rules (required documents, inspection 
procedures, approval thresholds).

Workaround: We're hardcoding "Contractor License" rules; planning to parameterize 
before production.

Needed before: March 15 (pilot go-live for first municipality)

Please clarify: Should license types be configuration, code, or database-driven?
```

---

### Severity 3: Normal (BACKGROUND)

**Definition:** Clarification is nice to have; implementation can proceed without delay.

**Examples:**
* "Enhancement request for future module"
* "Style preference question"
* "Process efficiency suggestion"
* "Module expansion idea"

**Response Time:** < 3 business days (best effort)

**Resolution Time:** < 1 week

**Escalation Method:**
1. Document the question (1-2 paragraphs)
2. State that this is not blocking
3. Email to info@publiclogic.com with subject: [NORMAL] [Issue Title]

**Examples of Normal Response:**
```
Subject: [NORMAL] Bulk Correction Process for TIME Module

Suggestion: For timesheets, staff sometimes need to correct past weeks 
(e.g., "last week I forgot to log 4 hours on Project X"). 

Current spec requires separate CASE per correction. Works but is verbose.

Question: Should we define a "bulk correction" sub-flow in v1.1 to group 
related corrections?

Not urgent; just asking for future planning.
```

---

## Response Time SLA Table

| Severity | Response Time | Resolution Time | Contact |
|----------|---|---|---|
| **Critical** | < 4 hours | < 24 hours | critical@publiclogic.com + email |
| **High** | < 24 hours | < 3 days | info@publiclogic.com |
| **Normal** | < 3 days | < 1 week | info@publiclogic.com |

---

## How to Escalate Correctly

### Format (All Escalations)

```
Subject: [SEVERITY] [Module] Brief Title

Context:
[1-2 sentences describing the situation]

Specific Question:
[What exactly do you need clarified?]

Current Approach:
[What workaround or assumption are you using?]

Impact:
[Why does this matter? What's blocked?]

Deadline:
[When do you need an answer? "Not urgent" is okay]

Code/Config Examples (if helpful):
[Snippet showing the ambiguity]
```

---

### Example: Well-Formatted Escalation

```
Subject: [CRITICAL] VAULTPRR Appeals Protocol - Case Reopening

Context:
VAULTPRR v1.0 defines closure as irreversible (no reopening). However, 
M.G.L. c. 66, §10(d) gives requester right to appeal within 90 days of closure 
to Supervisor. If appeal is granted, case must be reopened to fulfill Supervisor 
order.

Specific Question:
1. Should closed PRR CASE be reopened when appeal is granted?
2. Or should appeal result in a new separate CASE?
3. How should we track the relationship between original and appeal?

Current Approach:
We're currently treating appeals as new CASEs (appeal-001, appeal-002, etc.) 
separate from original request. But this breaks the audit trail and creates 
orphaned original records.

Impact:
Cannot deploy PRR to production until this is clear. Appeals are statutory right; 
must be handled correctly. Blocking 5 pilot municipalities.

Deadline:
ASAP (pilot go-live is Feb 15)

---

Thanks,
Polymorphic Engineering
```

---

## Common Escalation Scenarios

### Scenario 1: Statutory Interpretation

**You ask:** "Does M.G.L. c. 66, §10 allow partial disclosure if we can't find some records?"

**PublicLogic answers:** "[Legal analysis] Yes, if you document that you searched in all likely locations and found nothing."

**Resolution:** You proceed with confidence that interpretation is correct.

---

### Scenario 2: Module Interaction

**You ask:** "When ONBOARD closes, should system auto-create first TIME case?"

**PublicLogic answers:** "v1.0: No. Manual handoff. v1.1: We'll define 'workflow triggers' for this. For now, supervisor creates TIME case manually."

**Resolution:** You implement manual process; note as future enhancement.

---

### Scenario 3: Error Recovery

**You ask:** "If deadline computation fails, should we halt all transitions or use last-known-good deadline?"

**PublicLogic answers:** "Halt. Governance requires deadline enforcement; uncertain deadline is worse than no deadline. Follow Error_Handling_Recovery.md protocol."

**Resolution:** You implement halt strategy; alert ops; escalate to PublicLogic.

---

### Scenario 4: Config vs. Governance

**You ask:** "Can we set T10 to 15 days instead of 10 for 'slow' municipalities?"

**PublicLogic answers:** "No. M.G.L. c. 66, §10 specifies 10 days; you cannot change statute through config. If your municipality is slow, staff up earlier. If statute changes, we amend VAULT."

**Resolution:** You respect statutory limit; proceed with 10-day T10.

---

## Escalation Process Flowchart

```
Issue Identified
    ↓
Severity Assessment
    ├─ Blocks deployment? → CRITICAL (< 4h response)
    ├─ Blocks go-live? → HIGH (< 24h response)
    └─ Background improvement? → NORMAL (< 3d response)
    ↓
Document Clearly (use format above)
    ↓
Send to PublicLogic
    ├─ Critical: critical@publiclogic.com + info@publiclogic.com
    └─ High/Normal: info@publiclogic.com
    ↓
Receive Clarification or Amendment
    ├─ Clarification: Implement with guidance
    ├─ Amendment: Specification is updated; all deployments notified
    └─ Deferred: "v1.1 scope" — proceed with workaround
    ↓
Resume Implementation
    ↓
Document Decision (in code comments, commit message, etc.)
```

---

## What PublicLogic Will Do

### Within Response Time

1. Acknowledge receipt (within 2 hours for critical)
2. Ask clarifying questions if needed
3. Provide answer, workaround, or timeline

### Examples of Responses

**Quick Answer:**
```
Thanks for escalating. 

Answer: Yes, partial disclosure is allowed if you document your search. 
See Governance Architecture Appendix B for segregability rules.

Proceed with current approach.
```

**Workaround:**
```
This is a v1.1 feature (appeals automation). For v1.0:

Workaround: Treat appeal as new PRR case manually. When appeal is granted, 
create new CASE with note "Appeal to original case-001."

v1.1 will automate this.
```

**Amendment:**
```
This revealed an ambiguity in the spec. We're issuing VAULT v1.0.1 patch 
to clarify appeals handling. Deployment: by March 1.

Upgrade all active implementations from v1.0 to v1.0.1 before April 1.
```

---

## SLA Breaches

If PublicLogic misses an SLA:

1. **Critical miss:** Automatically escalate to PublicLogic leadership
2. **Notify:** "We missed SLA on your critical issue; here's our status"
3. **Compensate:** May offer expedited resolution or adjustment
4. **Document:** Breach is logged and analyzed for improvements

---

## Support Contact Info

**Urgent (Critical Severity):**
- Email: critical@publiclogic.com
- Phone: [PublicLogic emergency number]
- Response: < 4 hours

**Normal (High/Standard Severity):**
- Email: info@publiclogic.com
- Response: < 24 hours (High), < 3 days (Normal)

**Website:** www.publiclogic.com

---

## Questions About This SLA

If you don't understand severity tiers or how to escalate:
1. Email info@publiclogic.com with "[QUESTION]" in subject
2. Describe what you're unsure about
3. We'll clarify

Better to ask and be clear than to misclassify and create delays.

---

*SLAs ensure governance gaps do not block your work. Use them.*
