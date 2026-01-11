# VAULT Core Canon: Audit and Evidence Model

**Immutable Evidence Trail • Compliance and Discovery**

---

## Core Principle

VAULT maintains a **complete, immutable audit log** for every CASE.

This log answers three critical questions:
1. **What happened?** (what action was taken)
2. **Who did it?** (what actor)
3. **When?** (timestamp)
4. **Why?** (reason or rule applied)

---

## Audit Log Structure

Every CASE has an AuditLog array. Each entry is immutable (append-only).

```json
{
  "EventID": "evt-001",
  "Timestamp": "2026-01-15T14:32:00.000Z",
  "Actor": "rao@town.example.com",
  "ActorRole": "Records Access Officer",
  "Action": "Create",
  "StepID": "S300_RAO_Assessment",
  "ResourceType": "Asset",
  "ResourceID": "assess-001",
  "OldValue": null,
  "NewValue": {
    "AssetType": "Assessment",
    "Content": "Records are responsive; not exempt"
  },
  "Reason": "Initial records assessment per intake",
  "RuleApplied": null,
  "ErrorCode": null,
  "Evidence": {
    "AssetID": "assess-001"
  }
}
```

### Required Fields

| Field | Type | Purpose |
|-------|------|---------|
| **EventID** | UUID | Unique event identifier |
| **Timestamp** | DateTime UTC | Precise moment of action |
| **Actor** | String | Username or system ID |
| **ActorRole** | String | Job title or role (for context) |
| **Action** | Enum | Create, Update, Lock, Transition, Enforce, Error |
| **StepID** | String | Which step in process (e.g., S300_Assessment) |
| **ResourceType** | Enum | CASE, Asset, Timer, Gate, etc. |
| **ResourceID** | UUID | What was affected |
| **OldValue** | Any | Previous state (if Update) |
| **NewValue** | Any | New state (if Update or Create) |
| **Reason** | String | Why this action was taken |
| **RuleApplied** | String | If enforcement, which rule (e.g., "T10_Fee_Waiver") |
| **ErrorCode** | String | If error, error code |
| **Evidence** | Object | Supporting data (asset IDs, calculation details) |

---

## Common Audit Events

### Case Creation

```json
{
  "Action": "Create",
  "ResourceType": "CASE",
  "Reason": "PRR received via web form",
  "NewValue": {
    "CaseID": "prr-2026-001",
    "CaseType": "PRR",
    "Subject": "Jane Doe, email intake"
  }
}
```

### Asset Creation

```json
{
  "Action": "Create",
  "ResourceType": "Asset",
  "Reason": "Withholding letter drafted",
  "NewValue": {
    "AssetID": "asset-001",
    "AssetType": "WithholdingLetter",
    "RetentionClass": "KEEPER"
  }
}
```

### Asset Lock

```json
{
  "Action": "Lock",
  "ResourceType": "Asset",
  "ResourceID": "asset-001",
  "Reason": "Case closed; withholding letter is final",
  "Evidence": {
    "AssetType": "WithholdingLetter",
    "LockedTimestamp": "2026-01-15T16:45:00Z"
  }
}
```

### State Transition

```json
{
  "Action": "Transition",
  "ResourceType": "CASE",
  "OldValue": "Assessment",
  "NewValue": "Gathering",
  "Reason": "T25 forecast passed; records found to be responsive"
}
```

### Timer Enforcement

```json
{
  "Action": "Enforce",
  "ResourceType": "Timer",
  "ResourceID": "T10",
  "RuleApplied": "T10_Fee_Waiver",
  "Reason": "T10 deadline missed; response due 2026-01-20, sent 2026-01-25",
  "NewValue": {
    "FeesAllowed": false
  }
}
```

### Tolling Event

```json
{
  "Action": "Toll",
  "ResourceType": "Timer",
  "Reason": "Clarification requested; clock paused",
  "Evidence": {
    "TollingStart": "2026-01-20",
    "TollingReason": "Scope clarification needed",
    "TollingAsset": "CLARIFICATION_REQUEST"
  }
}
```

---

## Audit Trail Guarantees

### Immutability

Audit log entries **cannot be edited, deleted, or reordered** after creation.

Implementation options:
* Append-only database table (cannot UPDATE or DELETE)
* Immutable blob storage (e.g., object store with object lock)
* Event sourcing (immutable event stream)
* Blockchain or ledger

**All are valid** as long as entries cannot be mutated post-creation.

### Completeness

Every action must be logged:

| Action | Must Log? | Why |
|--------|-----------|-----|
| Create asset | YES | Evidence of creation |
| Update asset | YES | Evidence of change |
| Lock asset | YES | Evidence of finality |
| Transition state | YES | Process flow |
| Enforce rule | YES | Evidence of compliance |
| Error occurs | YES | Evidence of problem |

Missing an entry = audit failure.

### Ordering

Entries must be ordered by **Timestamp** (not by creation order in system).

If two events happen at the same millisecond, establish tie-breaking rule (e.g., by EventID).

### Queryability

Audit log must support queries:

* **By CaseID** — all events for a case
* **By Actor** — all events by an actor (for user accountability)
* **By Action** — all asset creations, all locks, etc.
* **By Date Range** — events between X and Y
* **By Rule** — all enforcement events for T10 rule

---

## Evidence and Discovery

VAULT audit logs support discovery and legal proceedings.

### What Auditors Can Verify

Given a CASE, auditor can confirm:

1. **Timeline is accurate** — every action has timestamp
2. **Authority is clear** — who did what and why
3. **Decisions are defensible** — supporting evidence is linked
4. **Process was followed** — required steps show in audit trail
5. **Deadlines were met** — or enforcement was applied

Example audit verification:

```
PRR Received: 2026-01-15 (Tuesday)
EffectiveReceiptDate: 2026-01-15
T10 DueDate: 2026-01-28 (10 business days)

Audit trail shows:
2026-01-15: Case created
2026-01-16: Assessment started
2026-01-20: Assessment completed; records found responsive
2026-01-21: Redactions applied
2026-01-22: Response letter drafted and locked
2026-01-28: Response letter delivered
2026-01-28: T10 met; case closed

Verification: PASS (deadline met)
```

### What's NOT Auditable

Some things do not appear in audit log:

* **Intent or reasoning** (unless explicitly recorded in "Reason" field)
* **Subjective judgments** (e.g., "This took longer because it was complex")
* **Unmade decisions** (e.g., "We almost applied this exemption but didn't")

Log what happened. Reason helps explain why. But complex rationale must be in case assets (memos, analysis).

---

## Retention of Audit Log

Audit logs are classified as **KEEPER** — permanent.

Minimum retention: 7 years from CASE closure per M.G.L. c. 66, §10.

Suggest: Permanent retention. Audit logs have low storage cost and high value.

---

## Reporting on Audit Data

VAULT must support these compliance reports:

### Case-Level Reports

* **Cases by status** — Open, closed, blocked
* **Cases by closure reason** — Delivered, withheld, timeout
* **Case aging** — How long open cases remain
* **Deadline compliance** — T10 met/missed, T25 met/missed

### Timeline Reports

* **Events by actor** — Who did what
* **Events by step** — Which process steps are slow
* **Events by rule** — Which enforcement rules trigger most

### Compliance Reports

* **T10 miss rate** — % of cases missing T10 deadline
* **T25 miss rate** — % of cases exceeding municipal limit
* **Fee waiver frequency** — How many times fee waiver triggered
* **Average case duration** — Mean time from intake to closure

---

## Audit Log as Evidence

If a case goes to litigation:

**VAULT audit log is discoverable.** Opposing counsel can request it.

**Advantage:** Audit log shows you followed process correctly.
**Disadvantage:** If you did NOT follow process, audit log proves it.

This is why doing things correctly matters. The audit log is evidence.

**Example — Good Case:**

```
Audit log shows:
✓ Intake recorded
✓ Deadlines computed correctly
✓ Assessment documented
✓ T25 gate passed
✓ Withholding decision reasoned
✓ Response delivered on time

Result: Auditor finds compliance.
```

**Example — Bad Case:**

```
Audit log shows:
✗ No intake record
✗ Deadlines not computed
✗ No assessment recorded
✗ Response sent after 45 days

Result: Auditor finds non-compliance.
```

VAULT prevents the second case.

---

## Breach of Audit Trail

What if someone edits an audit log entry?

### Detection

* Immutable storage makes this impossible (technically)
* Checksums or signatures can detect tampering
* Missing entries become gaps (visible in timeline)

### Response

If audit trail is compromised:

1. **Investigate** — How did tampering occur?
2. **Notify** — Legal, compliance, audit
3. **Remediate** — Restore from backup if available
4. **Report** — To Supervisor of Records and potentially law enforcement

**Rule:** Audit trail compromise is a serious event. Prevent it.

---

## Q&A

**Q: Can I delete old audit entries?**
A: No. Audit log is permanent. Storage is cheap; auditability is critical.

**Q: If I correct a CASE, does the audit show both versions?**
A: Yes. Original decision is locked; new version is created; both appear in audit trail with reasons.

**Q: Can someone hide their identity in the audit log?**
A: Not if you require authentication. Log the authenticated actor (not nickname).

**Q: Do I have to log every keystroke?**
A: No. Log actions (create, update, lock, transition). Not individual edits.

**Q: Can an actor request to remove themselves from audit log?**
A: No. Audit log is immutable and permanent.

---

*Audit log is how VAULT creates trustworthiness. If you do not maintain it carefully, VAULT loses value.*
