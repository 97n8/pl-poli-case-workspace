# VAULTPRR™ — Governance Architecture & Domain Model

**Framework-as-a-Standard™ • Version 1.0**  
**Authority:** PublicLogic™  
**Governing Law:** M.G.L. c. 66, §10 • 950 CMR 32.00

---

## Document Classification

| Attribute | Value |
|-----------|-------|
| **Type** | Governance Architecture |
| **Status** | Authoritative |
| **Confidentiality** | Internal / NDA Partners |
| **Owner** | PublicLogic™ |
| **Encoding Spec** | VAULTPRR_Encoding_Specification_v1.md |

---

## Purpose

VAULTPRR™ defines the **governed, enforcement-first operating system** for Massachusetts Public Records Requests.

This document converts statutory obligations into **structural requirements** so that:

1. **Deadlines are enforced, not tracked** — timers trigger consequences automatically
2. **Consequences are structural, not discretionary** — fee waivers execute without human judgment
3. **Records are immutable where required** — locked artifacts cannot be altered, only versioned
4. **Risk is carried by structure, not individuals** — operators execute within protected boundaries

**Core Principle:** If it isn't recordable, transferable, and defensible, it isn't safe.

---

## Document Hierarchy

VAULTPRR™ operates through three document tiers:

| Tier | Document | Purpose | Audience |
|------|----------|---------|----------|
| **1** | **Governance Architecture** (this document) | Statutory authority, enforcement principles, domain model | PublicLogic, executives, legal |
| **2** | **Encoding Specification** | Implementation contract — schemas, state machine, algorithms | Runtime partners (Polimorphic) |
| **3** | **Derivative Artifacts** | SOPs, training, UI specs, local policy | Municipal staff, end users |

**Compliance Rule:** Tier 2 implements Tier 1. Tier 3 layers on Tier 2. Conflicts resolve upward.

---

## What This Document Is

- Canonical governance architecture for Massachusetts PRR
- Statutory enforcement map with explicit legal anchors
- Domain model defining all entities and relationships
- Authority declaration establishing PublicLogic as governance owner
- Foundation for all encoding and implementation

## What This Document Is Not

- UI specification
- Training material
- Standard operating procedure
- Local policy or bylaw
- Encoding-level detail (see Encoding Specification)

---

## Part I: Statutory Authority

### Governing Law

VAULTPRR™ enforces — not merely references — the following statutory requirements:

#### M.G.L. c. 66, §10 — Public Records Law

| Requirement | Citation | VAULTPRR™ Enforcement |
|-------------|----------|----------------------|
| Initial written response within **10 business days** | §10(a) | T10 timer; automatic tracking |
| Municipal production limit: **25 business days** | §10(a) | T25 timer; extension gate |
| Extension requires **requester agreement** or **Supervisor grant** | §10(a) | Mandatory fork at T25 forecast |
| **Fees prohibited** if T10 deadline missed | §10(d) | System enforcement: `FeesAllowed = false` |
| Requester may petition Supervisor within **20 business days** | §10(b) | T20 window tracked |
| Withholding requires **written explanation** with exemption citations | §10(a) | Withholding Letter asset required |

#### 950 CMR 32.00 — Supervisor of Records Regulations

| Requirement | Citation | VAULTPRR™ Enforcement |
|-------------|----------|----------------------|
| Petition filing window: **20 business days** | 32.08(1) | T20 timer |
| Municipal extension limit: **30 business days from grant** | 32.06(2) | Extended T25 computation |
| Appeal window: **90 calendar days** | 32.08(2) | T90 timer (post-closure) |
| Records Access Officer designation required | 32.04 | RAO role in domain model |

### Statutory Interpretation Principles

1. **Deadlines are outer limits, not targets** — earlier response is always permissible
2. **Business days exclude weekends and legal holidays** — per M.G.L. c. 4, §7
3. **Receipt on non-business day shifts to next business day** — EffectiveReceiptDate computation
4. **Tolling requires documented pause** — clarification, requester delay, or Supervisor action
5. **Fee waiver is permanent once triggered** — T10 miss cannot be cured

---

## Part II: Enforcement Architecture

### Timer Model

All timers compute from **EffectiveReceiptDate**: the date the request is received, or the next business day if received on a weekend, legal holiday, or municipal closure.

| Timer | Definition | Consequence |
|-------|------------|-------------|
| **T10** | Initial written response due | If missed: `FeesAllowed = false` (permanent) |
| **T20** | Petition window closes | Requester loses right to petition Supervisor |
| **T25** | Municipal production limit | If forecast exceeds: must obtain agreement or file petition |
| **T90** | Appeal window (from closure) | Requester loses right to appeal |

### Automatic Enforcement Rules

These rules execute without human discretion:

| Rule | Trigger | Action |
|------|---------|--------|
| **Fee Waiver** | T10 deadline missed | Set `FeesAllowed = false`; log; no override permitted |
| **Extension Gate** | Forecast completion > T25 | Block progression until agreement or petition obtained |
| **Timeout Closure** | Clarification wait > 30 days | Close case as timeout; notify RAO |
| **Non-Payment Closure** | Payment wait > 30 days | Close case as non-payment; retain records |

### Tolling Protocol

The deadline clock may be paused (tolled) under specific conditions:

| Tolling Reason | Trigger | Resume Trigger |
|----------------|---------|----------------|
| **Clarification** | Municipality requests scope clarification | Requester responds |
| **Requester Delay** | Requester requests hold | Requester requests resumption |
| **Supervisor Extension** | Petition granted with extension | Extension period expires |

**Tolling Rules:**
- Tolling must be documented with start timestamp, reason, and trigger asset
- Resume must be documented with end timestamp and resume asset
- All deadlines recompute by adding tolled business days
- Tolling does not reset deadlines — it extends them

---

## Part III: Domain Model

### Entity Relationship Overview

```
CASE (root container)
├── RequesterIdentity
├── IntakeRecord
├── ScopeDefinition (versioned)
├── DeadlineModel
│   └── TollingEvent[]
├── CaseStatus
├── Asset[] (all documents)
│   ├── ExemptionTag[]
│   └── RedactionTag[]
└── LogEntry[] (audit trail)

PROCESS (execution layer)
├── StateID (current)
├── TransitionHistory
└── EnforcementFlags
```

### Entity Definitions

#### CASE

The root container for all PRR activity. One CASE per request.

| Attribute | Type | Description |
|-----------|------|-------------|
| CaseID | UUID | Immutable unique identifier |
| RequesterIdentity | Object | Who submitted the request |
| IntakeRecord | Object | How and when received |
| ScopeDefinition | Object | What records are requested (versioned) |
| DeadlineModel | Object | All statutory timers |
| FeesAllowed | Boolean | False if T10 missed |
| CaseStatus | Object | Current state and flags |
| ClosureReason | Enum | Outcome when closed |
| Assets | Array | All documents in the case |
| AuditLog | Array | Immutable activity record |

#### RequesterIdentity

| Attribute | Type | Description |
|-----------|------|-------------|
| Name | String | Requester name (may be anonymous) |
| Organization | String | Optional affiliation |
| Email | String | Contact email |
| Phone | String | Contact phone |
| MailingAddress | String | Physical address |
| ContactMethod | Enum | Preferred response delivery: EMAIL, MAIL, FAX, IN_PERSON |
| IsAnonymous | Boolean | MA PRR permits anonymous requests |

#### IntakeRecord

| Attribute | Type | Description |
|-----------|------|-------------|
| ReceivedTimestamp | DateTime | Actual receipt (UTC) |
| EffectiveReceiptDate | Date | Computed start for timers |
| Channel | Enum | WEB_FORM, EMAIL, MAIL, FAX, IN_PERSON, PHONE |
| RawRequestText | String | Verbatim request as submitted |
| Attachments | Array[UUID] | Asset references |

#### ScopeDefinition

| Attribute | Type | Description |
|-----------|------|-------------|
| Version | Integer | Incremented on each clarification |
| Description | String | Normalized scope statement |
| DateRangeStart | Date | Optional temporal bound |
| DateRangeEnd | Date | Optional temporal bound |
| Departments | Array[String] | Relevant departments |
| Keywords | Array[String] | Search terms |
| ClarificationHistory | Array | Prior versions with Q&A |

#### DeadlineModel

| Attribute | Type | Description |
|-----------|------|-------------|
| EffectiveReceiptDate | Date | Timer start date |
| T10 | Object | Due date, met flag, met timestamp |
| T20 | Object | Window close, petition filed flag |
| T25 | Object | Due date, extended date, extension source |
| T90 | Object | Window close (from closure), appeal filed flag |
| TollingEvents | Array | All pause/resume events |
| BusinessDaysTolled | Integer | Total tolled days |

#### Asset

| Attribute | Type | Description |
|-----------|------|-------------|
| AssetID | UUID | Unique identifier |
| AssetType | Enum | Document classification (see below) |
| RetentionClass | Enum | KEEPER, REFERENCE, TRANSACTIONAL |
| Locked | Boolean | Immutable after creation |
| CreatedTimestamp | DateTime | Creation time |
| CreatedBy | String | Actor identifier |
| VersionChain | Array[UUID] | Prior versions (for replacements) |
| FileReference | String | Storage location |
| Hash | String | SHA-256 integrity check |
| Exemptions | Array | Applied exemption tags |
| Redactions | Array | Applied redaction tags |

#### Asset Types

| Type | Description | Retention | Locked |
|------|-------------|-----------|--------|
| ORIGINAL_REQUEST | Verbatim request as received | KEEPER | Yes |
| CLARIFICATION_REQUEST | Municipality's clarification email | REFERENCE | No |
| CLARIFICATION_RESPONSE | Requester's clarification reply | REFERENCE | No |
| SCOPE_DOCUMENT | Finalized scope statement | KEEPER | No |
| SEARCH_LOG | Record of search activities | KEEPER | No |
| RESPONSIVE_RECORD | Original responsive document | REFERENCE | No |
| REDACTED_RECORD | Document with redactions applied | REFERENCE | No |
| EXEMPTION_LOG | Exemption determinations | KEEPER | No |
| NO_RECORDS_LETTER | Response: no responsive records | KEEPER | Yes |
| WITHHOLDING_LETTER | Response: full withholding with citations | KEEPER | Yes |
| RESPONSE_PACKAGE | Final response: letter + records + log | KEEPER | Yes |
| EXTENSION_AGREEMENT | Requester agreement to extended timeline | KEEPER | Yes |
| PETITION | Petition to Supervisor of Records | KEEPER | Yes |
| FEE_ESTIMATE | Itemized cost estimate | TRANSACTIONAL | No |
| INVOICE | Payment request | TRANSACTIONAL | No |
| PAYMENT_RECEIPT | Payment confirmation | TRANSACTIONAL | Yes |
| DELIVERY_CONFIRMATION | Proof of delivery | KEEPER | Yes |

#### LogEntry

| Attribute | Type | Description |
|-----------|------|-------------|
| LogID | UUID | Unique identifier |
| Timestamp | DateTime | Event time (UTC) |
| Actor | Object | Who performed action |
| Action | Enum | What happened |
| StateFrom | String | State before action |
| StateTo | String | State after action |
| AssetsAffected | Array[UUID] | Related assets |
| Metadata | Object | Additional context |

### Record Classes

| Class | Retention | Description |
|-------|-----------|-------------|
| **KEEPER** | Permanent | Authoritative records required for legal compliance and audit |
| **REFERENCE** | 7 years | Supporting documents with long-term relevance |
| **TRANSACTIONAL** | 1-2 years | Operational records with short-term relevance |

### Immutability Rules

| Rule | Application |
|------|-------------|
| **LOCKED on creation** | Original request, no records letter, withholding letter, response package, extension agreement, petition, payment receipt, delivery confirmation |
| **Versioned replacement only** | Locked assets cannot be edited; corrections require new version with chain reference |
| **Audit log is append-only** | Log entries cannot be modified or deleted |

---

## Part IV: Process Architecture

### State Categories

| Category | Description | Examples |
|----------|-------------|----------|
| **Form** | Data collection from user | Intake |
| **System** | Automated computation | Timer compute |
| **Review** | Human evaluation required | Assessment, exemption review |
| **Manual** | Human action required | Gather, redaction, delivery |
| **Document** | Asset creation | Response package |
| **Email** | Communication dispatch | Clarification request |
| **Wait** | Clock paused, awaiting response | Clarification wait, payment wait |
| **Gate** | Conditional progression | Counsel review |
| **Decision** | Fork based on evaluation | T25 forecast, fee assessment |
| **Terminal** | Case closed | Delivered, withheld, non-payment |

### Process Flow Summary

```
INTAKE → TIMER_COMPUTE → ASSESSMENT
                              │
              ┌───────────────┼───────────────┐
              ▼               ▼               ▼
        CLARIFICATION    NO_RECORDS      FORECAST_CHECK
              │               │               │
              ▼               │       ┌───────┴───────┐
           WAIT ─────────────►│       ▼               ▼
              │               │   AGREEMENT       PETITION
              ▼               │       │               │
         [resume] ───────────►│       └───────┬───────┘
                              │               ▼
                              │           GATHER
                              │               │
                              │               ▼
                              │       EXEMPTION_REVIEW
                              │               │
                              │       ┌───────┴───────┐
                              │       ▼               ▼
                              │   WITHHELD      REDACTION
                              │       │               │
                              │       │               ▼
                              │       │       COUNSEL_GATE
                              │       │               │
                              │       │               ▼
                              │       │          PACKAGE
                              │       │               │
                              │       │               ▼
                              │       │       FEE_ASSESSMENT
                              │       │               │
                              │       │       ┌───────┴───────┐
                              │       │       ▼               ▼
                              │       │   FEE_WAIVE       INVOICE
                              │       │       │               │
                              │       │       │               ▼
                              │       │       │       PAYMENT_WAIT
                              │       │       │               │
                              │       │       │       ┌───────┴───────┐
                              │       │       │       ▼               ▼
                              │       │       │   NON_PAYMENT     [paid]
                              │       │       │       │               │
                              │       │       └───────┴───────────────┤
                              │       │                               ▼
                              │       │                           DELIVERY
                              │       │                               │
                              ▼       ▼                               ▼
                           CLOSED ◄───────────────────────────────────┘
```

### Critical Gates

| Gate | Condition | Pass | Fail |
|------|-----------|------|------|
| **T25 Forecast** | Forecast completion ≤ T25 | Proceed to Gather | Require agreement or petition |
| **T10 Check** | Response by T10 due date | Fees permitted | `FeesAllowed = false` |
| **Counsel Review** | Legal approval (optional) | Proceed to Package | Return to Redaction |
| **Payment** | Payment received | Proceed to Delivery | Non-payment closure |

---

## Part V: Implementation Boundaries

### PublicLogic™ Responsibility

- Framework definition and governance authority
- Statutory interpretation and enforcement rules
- Domain model and entity definitions
- Process architecture and state definitions
- Compliance validation and audit requirements
- Change control and version management

### Runtime Partner Responsibility (Polimorphic)

- State machine implementation per Encoding Specification
- Data schema implementation per JSON schemas
- Timer computation per algorithms
- Tolling protocol per specification
- Error handling per defined codes
- Integration with external systems
- User interface (layered on top)

### Governance Gap Protocol

If encoding introduces ambiguity, dependency, or new risk:

1. **Halt** — deployment stops immediately
2. **Escalate** — partner notifies PublicLogic
3. **Resolve** — PublicLogic issues clarification or specification amendment
4. **Resume** — encoding continues with clarification

**Stop Rule:** If encoding introduces dependency or new risk, deployment halts.

---

## Part VI: Compliance Requirements

### Audit Trail Requirements

Every CASE must maintain an immutable audit log capturing:

- All state transitions with actor and timestamp
- All asset creation and locking events
- All timer computations and tolling events
- All enforcement rule executions
- All error conditions

### Retention Requirements

| Record Type | Minimum Retention |
|-------------|-------------------|
| CASE container | 7 years from closure |
| KEEPER assets | Permanent |
| REFERENCE assets | 7 years |
| TRANSACTIONAL assets | 2 years |
| Audit log | 7 years from closure |

### Reporting Requirements

System must support extraction of:

- Open cases by age and status
- T10/T25 compliance rates
- Fee waiver frequency and causes
- Closure outcomes by type
- Average response time by complexity

---

## Part VII: Change Control

### Amendment Authority

| Change Type | Authority | Process |
|-------------|-----------|---------|
| Statutory interpretation | PublicLogic | Legal review, version increment |
| Domain model change | PublicLogic | Architecture review, encoding spec update |
| Enforcement rule change | PublicLogic | Compliance review, implementation validation |
| Process flow change | PublicLogic | Architecture review, encoding spec update |

### Version History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2026-01-10 | PublicLogic | Initial release |

### Review Schedule

- **Quarterly:** Compliance validation against active cases
- **Annually:** Full architecture review
- **On demand:** Material law change, enforcement failure, audit finding

---

## Appendix A: Exemption Reference

Common Massachusetts PRR exemptions (non-exhaustive):

| Code | Citation | Description |
|------|----------|-------------|
| PRR-26a | M.G.L. c. 4, §7(26)(a) | Personnel, medical, or similar files |
| PRR-26b | M.G.L. c. 4, §7(26)(b) | Trade secrets, commercial information |
| PRR-26c | M.G.L. c. 4, §7(26)(c) | Investigatory materials |
| PRR-26d | M.G.L. c. 4, §7(26)(d) | Inter/intra-agency deliberative materials |
| PRR-26e | M.G.L. c. 4, §7(26)(e) | Notebooks of investigators |
| PRR-26f | M.G.L. c. 4, §7(26)(f) | Attorney-client privilege |
| PRR-26g | M.G.L. c. 4, §7(26)(g) | Social Security numbers |
| PRR-26n | M.G.L. c. 4, §7(26)(n) | Security measures |

**Segregability Rule:** If a record contains both exempt and non-exempt information, exempt portions must be redacted and non-exempt portions provided.

---

## Appendix B: Business Day Calendar

### Massachusetts Legal Holidays

Per M.G.L. c. 4, §7:

| Holiday | Date |
|---------|------|
| New Year's Day | January 1 |
| Martin Luther King Jr. Day | Third Monday, January |
| Presidents' Day | Third Monday, February |
| Patriots' Day | Third Monday, April |
| Memorial Day | Last Monday, May |
| Juneteenth | June 19 |
| Independence Day | July 4 |
| Labor Day | First Monday, September |
| Columbus Day | Second Monday, October |
| Veterans Day | November 11 |
| Thanksgiving | Fourth Thursday, November |
| Christmas | December 25 |

**Observed holidays:** When a holiday falls on Saturday, it is observed Friday. When it falls on Sunday, it is observed Monday.

**Municipal closures:** Additional non-business days (town meeting, emergency closure) must be configured per municipality.

---

## Appendix C: Glossary

| Term | Definition |
|------|------------|
| **Asset** | Any document or artifact within a CASE |
| **Business Day** | Weekday that is not a legal holiday or municipal closure |
| **CASE** | Root container for a single PRR |
| **EffectiveReceiptDate** | Computed date from which all timers start |
| **KEEPER** | Permanent retention class |
| **LOCKED** | Immutable; cannot be edited, only versioned |
| **RAO** | Records Access Officer — designated municipal official |
| **Supervisor** | Supervisor of Records — state oversight authority |
| **T10/T20/T25/T90** | Statutory deadline timers |
| **Tolling** | Pausing the deadline clock |

---

*This document is the authoritative governance architecture for VAULTPRR™. Implementation must conform to the companion Encoding Specification. Deviations require written approval from PublicLogic.*
