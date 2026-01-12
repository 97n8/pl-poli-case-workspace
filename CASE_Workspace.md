# VAULT Core Canon: CASE Workspace

**Universal Architecture • Applies to All Modules**

---

## What Is a CASE?

A CASE is the **root container for a single unit of work** in VAULT.

All authoritative facts, decisions, and artifacts for that work unit live **inside the CASE**.

Examples:

- One public records request = one CASE (VAULTPRR)
- One license application = one CASE (VAULT CLERK)
- One invoice = one CASE (VAULT FISCAL)
- One employee onboarding = one CASE (VAULT ONBOARD)
- One timesheet = one CASE (VAULT TIME)

**Core principle:** If it is not inside a CASE, it does not exist in VAULT.

---

## CASE Entity Model

Every CASE contains the following components.

### 1. Identity (immutable)

| Attribute | Type | Description |
|-----------|------|-------------|
| **CaseID** | UUID | Immutable unique identifier for this CASE |
| **CaseType** | Enum | PRR, ONBOARD, INVOICE, LICENSE, TIMESHEET, etc. |
| **CreatedTimestamp** | DateTime (UTC) | When the CASE was opened |
| **CreatedBy** | String | Actor who opened the CASE (role or username) |

**Rule:** Identity fields are treated as immutable after creation.

---

### 2. Subject (module-specific, indexable)

The CASE subject is the minimum set of fields required to understand what the CASE is about **without opening assets**.

Examples:

- **PRR:** RequesterIdentity, IntakeRecord
- **INVOICE:** Vendor, Amount, GLCode
- **ONBOARD:** Employee, Department, StartDate
- **LICENSE:** Applicant, ApplicationType, PropertyAddress

**Rule:** Subject must be sufficient for indexing, routing, and audit review without asset inspection.

---

### 3. Scope (versioned)

| Attribute | Type | Description |
|-----------|------|-------------|
| **ScopeDefinition** | Object | What is included in this CASE |
| **ScopeVersion** | Integer | Current version (starts at 1) |
| **ScopeHistory** | Array | Prior versions with timestamps and reasons |

**Rule:** Scope changes create a new version. Prior versions are preserved. Scope is not overwritten.

---

### 4. Deadlines (where applicable)

| Attribute | Type | Description |
|-----------|------|-------------|
| **Deadlines** | Object | All statutory or business deadlines |
| **TollingHistory** | Array | When the clock was paused and why |
| **EnforcementFlags** | Object | Rules triggered (e.g., FeesAllowed, ExtensionNeeded) |

**Modules that commonly include deadlines:** PRR, CLERK (some licenses), FISCAL (some invoices)  
**Modules that may not:** ONBOARD (typically single-pass)

---

### 5. Status and Gates

| Attribute | Type | Description |
|-----------|------|-------------|
| **CurrentState** | Enum | Current lifecycle state (e.g., OPEN, ASSESSMENT, GATHER, DELIVERY, CLOSED) |
| **ClosureReason** | Enum | Why the CASE was closed (e.g., Delivered, Withheld, Cancelled) |
| **ClosureTimestamp** | DateTime (UTC) | When the CASE closed (null if open) |
| **CanTransition** | Boolean | Whether the CASE can transition |
| **TransitionBlockers** | Array | Reasons the CASE cannot transition (e.g., “Payment not received”, “Counsel review pending”) |

**Rule:** Prior to transition, `TransitionBlockers` must be empty.

---

### 6. Assets (documents, decisions, artifacts)

| Attribute | Type | Description |
|-----------|------|-------------|
| **Assets** | Array | Documents, forms, letters, decisions, records, and artifacts |
| **AssetCount** | Integer | Count of assets |
| **TotalAssetSize** | Bytes | Storage size used |

Each Asset includes:



AssetID: UUID
AssetType: Enum (FormResponse, Letter, Decision, Record, etc.)
Filename: String
CreatedTimestamp: DateTime UTC
CreatedBy: String
RetentionClass: Enum (KEEPER, REFERENCE, TRANSACTIONAL)
IsLOCKED: Boolean
LOCKEDTimestamp: DateTime UTC
VersionChain: Array (all versions, newest first)
Tags: Array (Exemption, Redaction, etc.)


**Rule:** When an asset becomes LOCKED, it is immutable. Corrections are issued as new versions.

---

### 7. Audit Log (append-only)

| Attribute | Type | Description |
|-----------|------|-------------|
| **AuditLog** | Array | Append-only event history for the CASE |
| **EventCount** | Integer | Count of events recorded |

Each LogEntry contains:



EventID: UUID
Timestamp: DateTime UTC
Actor: String
Action: Enum (Create, Update, LOCK, Transition, Enforce, Error)
StepID: String
AssetIDs: Array
OldValue: Any
NewValue: Any
Reason: String
RuleApplied: String
ErrorCode: String
Evidence: Any


**Rule:** AuditLog is append-only. Entries are not deleted or edited.

---

## CASE Lifecycle



CREATE → CASE opens, ID assigned, subject recorded, deadlines computed (if applicable)

OPEN → Work occurs, assets created, decisions recorded

DECISION → Required gates evaluated (e.g., T25 forecast, exemption review)

CLOSURE → Gates satisfied, closure reason recorded, closure validation completed

LOCKED → CASE becomes immutable and audit trail is sealed

RETAINED → CASE stored per retention requirements

DESTROYED → CASE destroyed at end of retention (per policy)


**Lifecycle rules:**
- Transitions move forward only
- Transitions require no blockers
- Closure is irreversible in v1.0 (no reopen)

---

## What Happens Inside a CASE (Examples)

### Asset Creation

When work is performed, an asset is created:



Step: Assessment
Asset Created: ASSESSMENT_DECISION (AssetType: Decision)
Content: "Records are responsive and not exempt"
CreatedBy: "rao@town.example.com
"
IsLOCKED: false
RetentionClass: KEEPER
Audit: "Decision recorded"


### Asset Locking

When an output is final, it is LOCKED:



Asset: WITHHOLDING_LETTER
IsLOCKED: true
LOCKEDBy: "rao@town.example.com
"
LOCKEDTimestamp: "2026-01-15T14:32:00Z"
Reason: "Final determination"


Once LOCKED, modifications are represented as a new version in the version chain.

### Decision Gates

When a gate is evaluated:



Gate: T25 Forecast
Result: FAIL (completion > 25 days)
Action: Extension Required
EnforcementFlag: ExtensionNeeded = true
TransitionBlocker: "Extension agreement or petition required"
Audit: "T25 gate failed; extension required"


### Timer Enforcement

When a deadline is missed:



Timer: T10
DueDate: 2026-01-20
ActualResponse: 2026-01-25
Result: MISS
EnforcementFlag: FeesAllowed = false
Audit: "T10 missed; fees not permitted under governing rule"


### Closure

When all gates pass:



State: OPEN → CLOSED
ClosureReason: Delivered
ClosureTimestamp: 2026-01-25T16:45:00Z
AssetsLOCKED: 5
AuditEntries: 23
RetentionClass: KEEPER


---

## CASE Invariants (Non-Negotiable)

1. **One ID per CASE** — `CaseID` is unique and immutable
2. **One work unit per CASE** — do not combine distinct work units
3. **One closure per CASE** — closure is final in v1.0
4. **All assets inside** — authoritative documents live in the CASE
5. **All decisions recorded** — decisions generate audit entries
6. **No mutation after LOCK** — LOCKED assets are immutable
7. **Timers are enforced** — deadlines trigger consequences
8. **Audit trail is complete** — every action is logged

---

## CASE in Different Modules (Illustrative)

### VAULTPRR (Public Records Request)



CaseID: prr-2026-001
CaseType: PRR
Subject: RequesterIdentity (Jane Doe), IntakeChannel (email)
Scope: "All correspondence regarding Zoning Board variance decision"
ScopeVersion: 2
Deadlines: T10=2026-01-20, T25=2026-02-03, T90=2026-05-01
State: DELIVERY
Assets: [Intake Form, Assessment, Search Memo, Withholding Letter, Approved Records, Delivery Confirmation]
AuditLog: [23 entries]
ClosureReason: Delivered


### VAULT CLERK (License)



CaseID: clerk-2026-0145
CaseType: LICENSE
Subject: Applicant (ABC Plumbing LLC), ApplicationType (Contractor)
Scope: "Annual contractor license renewal"
ScopeVersion: 1
Deadlines: T10=2026-02-10
State: INSPECTION
Assets: [Application, Proof of Insurance, Background Check, Inspection Form]
AuditLog: [12 entries]


### VAULT FISCAL (Invoice)



CaseID: fiscal-2026-0892
CaseType: INVOICE
Subject: Vendor (MainTech Services), Amount ($4,500), GLCode (4100.001)
Scope: "Professional services through 2026-01-31"
ScopeVersion: 1
Deadlines: T10=2026-02-15
State: PAID
Assets: [Invoice, PO, Receipt, Check Register Entry, Payment Confirmation]
AuditLog: [8 entries]
ClosureReason: Paid


---

## Required Query Support

VAULT implementations must support:

**Across CASEs**
- By `CaseID` (exact)
- By `CaseType`
- By `CurrentState`
- By `CreatedTimestamp` range
- By `ClosureReason`

**Within a CASE**
- Assets by `AssetType`
- Audit entries by `Actor`
- Enforcement flags and applied rules
- Timeline of transitions

---

## Closure Validation Checklist

Prior to closure:

- [ ] `CaseID` and `CaseType` present and immutable
- [ ] Subject recorded
- [ ] Scope final, or version history justified
- [ ] Deadlines computed (if applicable)
- [ ] Required gates evaluated
- [ ] No transition blockers remain
- [ ] Closure reason set
- [ ] Required assets LOCKED
- [ ] Audit log complete and sealed
- [ ] Retention class assigned

---

*CASE is the common substrate across all modules. If anything here is ambiguous during implementation, please flag it early so it can be clarified before runtime decisions are locked.*
