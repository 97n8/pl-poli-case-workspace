# VAULT PRR Module — Charter

**Public Records Requests • Massachusetts Municipal Governance**

---

## Module Purpose

VAULT PRR enforces M.G.L. c. 66, §10 (Public Records Law) and 950 CMR 32.00 (Supervisor Regulations) as an integrated CASE-based system.

This module defines the complete PRR lifecycle:
1. **Intake** — Request received and logged
2. **Assessment** — Scope clarified, timeline computed, responsiveness determined
3. **Gathering** — Records located and retrieved
4. **Review** — Exemptions identified and applied
5. **Response** — Records delivered or withheld with reasoning
6. **Closure** — CASE closed with evidence locked
7. **Appeal** — (Post-closure) Supervisor appeal window open

---

## Statutory Authority

### Primary Statutes

**M.G.L. c. 66, §10 — Public Records Law**
* 10 business day initial response (T10)
* 25 business day production limit (T25)
* 20 business day petition window (T20)
* 90 calendar day appeal window (T90)
* Fee prohibition if T10 missed
* Requester right to petition Supervisor

**950 CMR 32.00 — Supervisor of Records Regulations**
* Supervisor authority to grant extensions
* Petition procedures and timelines
* Appeal procedures and standards
* Records Access Officer designation requirements

### Exemptions

VAULTPRR applies M.G.L. c. 4, §7(26) exemptions (trade secrets, personnel, investigatory, deliberative, etc.).

Full exemption reference in Appendix of Governance Architecture.

---

## CASE Model for PRR

### Identity

```json
{
  "CaseID": "prr-2026-001",
  "CaseType": "PRR",
  "CreatedTimestamp": "2026-01-15T09:30:00Z",
  "CreatedBy": "intake@town.example.com"
}
```

### Subject

```json
{
  "RequesterIdentity": {
    "Name": "Jane Doe",
    "Organization": null,
    "Email": "jane@example.com",
    "Phone": "555-0100",
    "ContactMethod": "EMAIL"
  },
  "IntakeRecord": {
    "ReceivedTimestamp": "2026-01-15T09:15:00Z",
    "EffectiveReceiptDate": "2026-01-15",
    "Channel": "WEB_FORM",
    "RawRequestText": "All correspondence regarding Zoning Board meeting on Jan 15, 2026"
  }
}
```

### Scope (Versioned)

```json
{
  "ScopeDefinition": {
    "Description": "All correspondence regarding Zoning Board meeting on Jan 15",
    "DateRange": ["2026-01-01", "2026-01-31"],
    "RecordTypes": ["Email", "Memo"],
    "Participants": ["Board Chair", "Town Counsel"]
  },
  "ScopeVersion": 1,
  "ScopeHistory": []
}
```

If requester clarifies: Create ScopeVersion 2 with reason "Clarification: Limit to board meeting only, not all correspondence."

### Deadlines

```json
{
  "T10": {
    "DueDate": "2026-01-28",
    "Status": "MET" | "OPEN" | "MISSED"
  },
  "T20": {
    "DueDate": "2026-02-06",
    "Status": "OPEN" | "CLOSED"
  },
  "T25": {
    "DueDate": "2026-02-19",
    "Status": "FORECAST_OK" | "EXTENSION_NEEDED"
  },
  "T90": {
    "DueDate": null,
    "Status": "NOT_APPLICABLE" (until case closed, then computes)
  },
  "TollingHistory": []
}
```

### Status and Gates

```json
{
  "CurrentState": "ASSESSMENT" | "GATHERING" | "REVIEW" | "DELIVERY" | "CLOSED",
  "ClosureReason": "Delivered" | "Withheld" | "NoRecords" | "Timeout" | "NonPayment" | null,
  "ClosureTimestamp": null,
  "CanTransition": true | false,
  "TransitionBlockers": []
}
```

### Assets (Examples)

```json
{
  "Assets": [
    {
      "AssetID": "asset-001",
      "AssetType": "IntakeForm",
      "RetentionClass": "REFERENCE",
      "IsLocked": false
    },
    {
      "AssetID": "asset-002",
      "AssetType": "AssessmentDecision",
      "RetentionClass": "KEEPER",
      "IsLocked": false
    },
    {
      "AssetID": "asset-003",
      "AssetType": "WithholdingLetter",
      "RetentionClass": "KEEPER",
      "IsLocked": false
    },
    {
      "AssetID": "asset-004",
      "AssetType": "ApprovedRecords",
      "RetentionClass": "KEEPER",
      "IsLocked": false
    }
  ]
}
```

---

## Module-Specific Rules

### Clarification Tolling

If RAO requests clarification:

* Clock is paused
* Requester has 30 days to respond
* If no response in 30 days: CASE closes as TIMEOUT
* If response received: Clock resumes, deadlines recompute

Tolling is logged.

### T25 Extension Gate

At T25 due date:
* If completion estimated ≤ T25: Proceed to gathering
* If completion estimated > T25: CASE blocks
  * Requester must agree to extension, OR
  * RAO must file petition with Supervisor

Without agreement or petition: CASE cannot transition past Assessment.

### Fee Enforcement

**T10 Compliance Check:** Before generating invoice
* If T10 due date has passed AND response not sent: Set `FeesAllowed = false`
* If FeesAllowed = false: Do not calculate or charge fees

**Permanent Rule:** Once T10 is missed, FeesAllowed remains false for CASE lifetime. Cannot be "cured."

### Appeal Window

After CASE is closed:
* T90 timer activates
* Requester may appeal to Supervisor within 90 calendar days
* After T90: Appeal right is lost

**Note:** Appeals are v1.1 scope. In v1.0, appeal notifications are sent but cases do not reopen.

---

## Asset Types in PRR

| Asset Type | Retention | Locked at Closure? | Purpose |
|------------|-----------|-------------------|---------|
| IntakeForm | REFERENCE | No | Record of how request came in |
| ClarificationRequest | REFERENCE | No | If clarification was needed |
| ClarificationResponse | REFERENCE | No | Requester's response to clarification |
| AssessmentDecision | KEEPER | Yes | RAO's initial responsiveness assessment |
| SearchMemo | REFERENCE | No | Where records were found |
| ExemptionAnalysis | KEEPER | Yes | Detailed exemption review |
| WithholdingLetter | KEEPER | Yes | If records withheld, reason |
| ApprovedRecords | KEEPER | Yes | Records being released |
| DeliveryConfirmation | KEEPER | Yes | Evidence delivery was made |
| FeesEstimate | REFERENCE | No | If fees charged |
| FeesInvoice | TRANSACTIONAL | No | Billing record |
| PaymentConfirmation | TRANSACTIONAL | No | If fees were paid |

---

## Process Flow (High Level)

```
Intake → Assessment → [Clarification?] → Gathering → Review → 
  [Fee Check] → [Payment Wait?] → Delivery → Closure
```

**Detailed process** is in VAULTPRR_Process_Flowchart_v1.md.

---

## Governance Gap Protocol for PRR

If you encounter ambiguity while implementing:

**Examples:**

* "What counts as initial 'response' if we send partial records?"
* "Can we combine T25 deadline extension request with petition?"
* "How do we handle requests for records that don't exist at intake but are created during T25?"

**Process:**

1. Document the ambiguity clearly
2. Send to info@publiclogic.com
3. PublicLogic responds with clarification or spec amendment
4. Resume implementation

**Do not guess at governance.** Escalate.

---

## Compliance Checklist for PRR CASE

Before closing a PRR CASE:

**Intake**
- [ ] IntakeForm recorded
- [ ] EffectiveReceiptDate computed
- [ ] T10, T20, T25 computed

**Assessment**
- [ ] Scope is clear (version 1 or latest)
- [ ] Responsiveness determined (records exist? location known?)
- [ ] T25 forecast evaluated (extension needed? gate passed?)

**Gathering & Review**
- [ ] Search memo completed (where found)
- [ ] Exemption analysis completed (which exemptions apply?)
- [ ] Redactions applied (if partial disclosure)
- [ ] Legal review done (if required)

**Response**
- [ ] Response letter drafted (approval or withholding)
- [ ] T10 deadline evaluated (if missed, FeesAllowed = false)
- [ ] If fees: Invoice generated and payment received

**Closure**
- [ ] Delivery confirmed (delivery date recorded)
- [ ] All key assets locked (letters, records, approvals)
- [ ] Audit log sealed
- [ ] ClosureReason set (Delivered, Withheld, etc.)
- [ ] T90 timer activated (appeal window)

---

## What You Will Find in This Module

1. **Control_Flow.mmd** — Detailed state machine with all transitions
2. **Response_Assembly.mmd** — Sub-flow for how response is built
3. **Timers_and_States.mmd** — Timer computations and state guards
4. **Decision_Tables.md** — Decision gates and outcomes
5. **Artifacts.md** — Asset types and retention rules

---

*VAULT PRR is the reference implementation. Other modules follow this same pattern and structure. Understand PRR fully before implementing other modules.*
