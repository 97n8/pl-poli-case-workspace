# VAULT Core Canon: CASE Workspace

**Universal Architecture • All Modules**

---

## What is a CASE?

A CASE is the **root container for a single work unit** in VAULT.

Everything that matters about that work unit lives inside the CASE and nowhere else.

Examples:
* One public records request = one CASE (VAULTPRR)
* One license application = one CASE (VAULT CLERK)
* One invoice = one CASE (VAULT FISCAL)
* One employee = one CASE (VAULT ONBOARD)
* One timesheet = one CASE (VAULT TIME)

**Core Principle:** If it is not inside a CASE, it does not exist in VAULT.

---

## CASE Entity Model

Every CASE contains:

### 1. Identity

| Attribute | Type | Description |
|-----------|------|-------------|
| **CaseID** | UUID | Immutable unique identifier for this CASE |
| **CaseType** | Enum | PRR, ONBOARD, INVOICE, LICENSE, TIMESHEET, etc. |
| **CreatedTimestamp** | DateTime UTC | When CASE was opened |
| **CreatedBy** | String | Actor who opened the CASE (role or username) |

**Immutable after creation.** Never change CaseID or CaseType.

---

### 2. Subject (varies by module)

Examples:

**PRR:** RequesterIdentity, IntakeRecord
**INVOICE:** Vendor, Amount, GLCode
**ONBOARD:** Employee, Department, StartDate
**LICENSE:** Applicant, ApplicationType, PropertyAddress

**Rule:** Must be sufficient to understand what the CASE is about without opening assets.

---

### 3. Scope (versioned)

| Attribute | Type | Description |
|-----------|------|-------------|
| **ScopeDefinition** | Object | What is included in this CASE |
| **ScopeVersion** | Integer | Current version (starts at 1) |
| **ScopeHistory** | Array | All previous versions with timestamps and reasons |

**Rule:** If scope changes (e.g., PRR requester clarifies or expands request), create a new version. Do not overwrite.

---

### 4. Deadlines (if applicable)

| Attribute | Type | Description |
|-----------|------|-------------|
| **Deadlines** | Object | All statutory or business deadlines |
| **TollingHistory** | Array | When clock was paused and why |
| **EnforcementFlags** | Object | Which rules have triggered (e.g., FeesAllowed, ExtensionNeeded) |

**Modules that have deadlines:** PRR, CLERK (some licenses), FISCAL (some invoices)

**Modules that do not:** ONBOARD (single-pass process)

---

### 5. Status and Gates

| Attribute | Type | Description |
|-----------|------|-------------|
| **CurrentState** | Enum | Where in the process this CASE is (e.g., OPEN, ASSESSMENT, GATHER, DELIVERY, CLOSED) |
| **ClosureReason** | Enum | Why CASE was closed (e.g., Delivered, Withheld, Cancelled) |
| **ClosureTimestamp** | DateTime UTC | When CASE was closed (null if open) |
| **CanTransition** | Boolean | Whether this CASE can move to the next state |
| **TransitionBlockers** | Array | What is preventing transition (e.g., ["Payment not received", "Counsel review pending"]) |

**Rule:** Before transition, all TransitionBlockers must be empty.

---

### 6. Assets (all documents, decisions, artifacts)

| Attribute | Type | Description |
|-----------|------|-------------|
| **Assets** | Array | All documents, forms, letters, decisions, etc. in this CASE |
| **AssetCount** | Integer | How many assets |
| **TotalAssetSize** | Bytes | Storage used |

**Each Asset has:**
```
AssetID: UUID
AssetType: Enum (FormResponse, Letter, Decision, Record, etc.)
Filename: String
CreatedTimestamp: DateTime UTC
CreatedBy: String
RetentionClass: Enum (KEEPER, REFERENCE, TRANSACTIONAL)
IsLocked: Boolean (cannot be mutated if true)
LockedTimestamp: DateTime UTC (when locked)
VersionChain: Array (all versions, newest first)
Tags: Array (Exemption, Redaction, etc.)
```

**Rule:** Once an asset is LOCKED, it cannot be edited. New versions must be created.

---

### 7. Audit Log (immutable)

| Attribute | Type | Description |
|-----------|------|-------------|
| **AuditLog** | Array | All events in this CASE |
| **EventCount** | Integer | How many events recorded |

**Each LogEntry contains:**
```
EventID: UUID
Timestamp: DateTime UTC (precise)
Actor: String (who made the change)
Action: Enum (Create, Update, Lock, Transition, Enforce, Error)
StepID: String (which step in process)
AssetIDs: Array (which assets affected)
OldValue: Any (what changed from)
NewValue: Any (what changed to)
Reason: String (why)
RuleApplied: String (if enforcement, which rule)
ErrorCode: String (if error, code)
Evidence: Any (supporting data)
```

**Rule:** Audit log is immutable. New entries are appended, never deleted or edited.

---

## CASE Lifecycle

```
1. CREATE      → CASE opens, ID assigned, subject recorded, deadlines computed
2. OPEN        → Work happens, assets created, decisions made
3. DECISION    → Critical gates are evaluated (e.g., T25 forecast, exemption review)
4. CLOSURE     → All gates passed, CASE validated, closure reason recorded
5. LOCKED      → CASE is immutable, audit trail is sealed
6. RETAINED    → CASE sits in archive per retention class
7. DESTROYED   → At end of retention, destroyed per policy
```

**Transition rules:**
* Can only move forward
* Cannot transition if blockers exist
* Closure is irreversible (no reopening in v1.0)

---

## What Happens Inside a CASE

### Assets Are Created

When work is done (form filled, letter drafted, decision made), an asset is created:

```
Step: Assessment
Asset Created: ASSESSMENT_DECISION (asset type: Decision)
  Content: "Records are responsive and not exempt"
  CreatedBy: "rao@town.example.com"
  Locked: false
  RetentionClass: KEEPER
Log Entry: "Decision recorded"
```

### Assets Are Locked

When work is final, the asset is locked:

```
Asset: WITHHOLDING_LETTER
Status: Locked
LockedBy: "rao@town.example.com"
LockedTimestamp: "2026-01-15T14:32:00Z"
Reason: "Withholding decision is final per M.G.L. c. 66"
```

Once locked, it cannot be edited. Corrections create a new version.

### Decisions Are Made

When a gate is evaluated:

```
Gate: T25 Forecast
Result: FAIL (completion > 25 days)
Action: Extension Required
EnforcementFlag: ExtensionNeeded = true
TransitionBlocker: "Extension agreement or petition required"
Log Entry: "T25 gate failed; extension required"
```

### Timers Are Enforced

When a deadline is missed:

```
Timer: T10
DueDate: 2026-01-20
ActualResponse: 2026-01-25
Result: MISS
EnforcementFlag: FeesAllowed = false
Log Entry: "T10 deadline missed; fees waived per M.G.L. c. 66, §10(d)"
```

### CASE Is Closed

When all gates pass:

```
Status: OPEN → CLOSED
ClosureReason: Delivered
ClosureTimestamp: 2026-01-25T16:45:00Z
Assets Locked: 5 (all response and delivery records)
AuditLog Sealed: 23 entries
RetentionClass: KEEPER (7 years minimum)
```

---

## CASE Invariants (Rules That Never Change)

1. **One ID per CASE** — CaseID is immutable and unique
2. **One subject per CASE** — Do not mix work units; create separate CASEs
3. **One closure per CASE** — Once closed, cannot reopen (v1.0)
4. **All assets inside** — No documents exist outside CASE boundaries
5. **All decisions recorded** — Every choice gets an audit entry
6. **No mutation after lock** — Locked assets cannot be edited
7. **Timers are enforced** — Deadlines trigger consequences, not reminders
8. **Audit trail is complete** — Every action creates an entry

---

## CASE in Different Modules

### VAULTPRR (Public Records Request)

```
CaseID: prr-2026-001
CaseType: PRR
Subject: RequesterIdentity (Jane Doe), IntakeChannel (email)
Scope: "All correspondence regarding Zoning Board variance decision"
ScopeVersion: 2 (requester clarified initially)
Deadlines: T10=2026-01-20, T25=2026-02-03, T90=2026-05-01
Status: DELIVERY
Assets: [Intake Form, Assessment, Search Memo, Withholding Letter, Approved Records, Delivery Confirmation]
AuditLog: [23 entries]
ClosureReason: Delivered (when delivery confirmed)
```

### VAULT CLERK (License)

```
CaseID: clerk-2026-0145
CaseType: LICENSE
Subject: ApplicantIdentity (ABC Plumbing LLC), ApplicationType (Contractor)
Scope: "Annual contractor license renewal"
ScopeVersion: 1
Deadlines: T10=2026-02-10 (municipal review window)
Status: INSPECTION
Assets: [Application, Proof of Insurance, Background Check, Inspection Form]
AuditLog: [12 entries]
ClosureReason: Pending (still open)
```

### VAULT FISCAL (Invoice)

```
CaseID: fiscal-2026-0892
CaseType: INVOICE
Subject: Vendor (MainTech Services), Amount ($4,500), GLCode (4100.001)
Scope: "Professional services through 2026-01-31"
ScopeVersion: 1
Deadlines: T10=2026-02-15 (payment terms)
Status: PAID
Assets: [Invoice, PO, Receipt, Check Register Entry, Payment Confirmation]
AuditLog: [8 entries]
ClosureReason: Paid
```

---

## CASE Queries

VAULT systems must support these queries:

**Find all CASEs:**
* By CaseID (exact)
* By CaseType (all PRRs, all invoices)
* By Status (open, closed, blocked)
* By CreatedDate (range)
* By ClosureReason (breakdown)

**Within a CASE:**
* Get all assets of type X
* Get all log entries by actor Y
* Get all enforced rules applied
* Get timeline of state transitions

---

## CASE Compliance Checklist

Before closing a CASE:

- [ ] CaseID is valid and immutable
- [ ] CaseType is set and immutable
- [ ] Subject is recorded
- [ ] Scope is final (or justified as versioned)
- [ ] All deadlines have been computed
- [ ] All assets are recorded inside CASE
- [ ] All critical gates have been evaluated
- [ ] No transition blockers remain
- [ ] Closure reason is set
- [ ] All LOCKED assets are immutable
- [ ] Audit log is complete and sealed
- [ ] Retention class is set

---

*CASE Workspace is the foundation of all VAULT modules. If you do not understand CASE, you cannot implement VAULT. Re-read until it is clear.*
