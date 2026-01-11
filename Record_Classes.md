# VAULT Core Canon: Record Classes and Immutability

**How Records Are Protected • Retention and Mutation Rules**

---

## Record Classification

Every asset in VAULT is assigned a **retention class** that determines:
* How long it is kept
* Whether it can be edited
* When it is destroyed
* What happens to it during audit

Three classes exist:

### 1. KEEPER (Permanent)

**Retention:** Permanent (indefinite)

**When to use:**
* Statutory decisions (withholding letters, assessments)
* Legal determinations (exemption findings)
* Formal approvals and denials
* Audit evidence
* Governance records

**Examples:**
* PRR withholding letter (evidence of decision)
* License approval or denial
* Invoice approval (audit trail for spending)
* Board meeting minutes
* Signed contracts
* Payroll records

**Mutation rules:** KEEPER assets may be edited while CASE is OPEN. Once CASE is CLOSED, must be LOCKED (immutable).

**Destruction:** Normally never. Check statute and legal hold requirements.

---

### 2. REFERENCE (7-Year Minimum)

**Retention:** 7 years from CASE closure

**When to use:**
* Supporting documents that explain decisions
* Search memos and research
* Correspondence with requester/applicant
* Internal notes and analysis
* Draft versions of final documents

**Examples:**
* PRR search memo (showed where records were found)
* License inspection notes
* Invoice processing checklist
* Staff communication about a case
* Rejected PO or draft approval

**Mutation rules:** REFERENCE assets may be edited while CASE is OPEN. Once CASE is CLOSED, may remain mutable (for corrections) but all changes audit-logged.

**Destruction:** Destroy after 7 years, unless legal hold or special statute applies.

---

### 3. TRANSACTIONAL (1-2 Year Minimum)

**Retention:** 1-2 years from CASE closure

**When to use:**
* Temporary working documents
* Intermediate processing steps
* Status updates
* Comments and notes that do not affect decision

**Examples:**
* Checkbox on form ("scanned page 3")
* Status log ("awaiting payment")
* Partial scan of record before assembly
* Internal progress note

**Mutation rules:** TRANSACTIONAL assets may be edited or deleted while CASE is OPEN. Once CASE is CLOSED, should be considered read-only (but not formally locked).

**Destruction:** May be destroyed after 1-2 years if no legal hold.

---

## How to Classify a Record

Ask these questions in order:

1. **Is this a statutory decision or formal approval?**
   → KEEPER

2. **Is this part of explaining how a decision was made?**
   → REFERENCE

3. **Is this temporary working material with no permanent value?**
   → TRANSACTIONAL

---

## Mutation Rules

### While CASE Is OPEN

All assets may be edited:

```
Step: Review
Asset: EXEMPTION_ANALYSIS
Status: OPEN, not locked
Action: RAO finds additional exemption
Content: ["Trade Secret (§26b)", "Deliberative (§26d)"]
Edited: "2026-01-15 14:32"
LogEntry: "Exemption analysis updated"
```

This is normal. Work is in progress.

---

### At CASE Closure

**Decision Point:** Does this asset need to be LOCKED?

| Asset Type | Locked? | Reason |
|------------|---------|--------|
| Withholding Letter | YES | Evidence of decision; must not change |
| License Approval | YES | Official record; must not change |
| Invoice Approval | YES | Audit trail; must not change |
| Search Memo | NO | Supporting; can be updated for clarity |
| Processing Checklist | NO | Working; can be updated |

**Rule:** Assets that are the evidence of a decision → LOCKED
Assets that support the decision → remain mutable (but logged)

---

### After CASE Is Closed and Assets Are LOCKED

LOCKED assets cannot be edited. Period.

If correction is needed:

```
Original Asset: ASSESSMENT_DECISION
Status: LOCKED (created 2026-01-15)
Content: "Records are not responsive"

Correction Needed: "Actually, records ARE responsive"

Action: 
1. Create new asset: ASSESSMENT_DECISION_v2
2. Set as locked
3. Add LogEntry: "Original assessment corrected"
4. Link assets as version chain

Asset Chain:
├── ASSESSMENT_DECISION_v1 (LOCKED, 2026-01-15)
└── ASSESSMENT_DECISION_v2 (LOCKED, 2026-01-16) [supersedes v1]
```

Both versions exist. The chain is visible.

---

## Immutability Model

### What "LOCKED" Means

A LOCKED asset has three properties:

1. **Cannot be edited** — The content cannot be changed
2. **Timestamp is recorded** — When it was locked is recorded
3. **Version chain is maintained** — If changed, previous versions are visible

### Why LOCKED Exists

LOCKED protects evidence.

When you tell an auditor "This decision was made on this date, in this way," the locked asset is proof.

If the asset could be edited, the auditor cannot trust it.

### How LOCKED Is Implemented

**Storage level:** You choose. Examples:

* Database column `locked = true` prevents UPDATE
* Immutable blob storage (e.g., append-only log)
* Document versioning with lock flag
* Blockchain or event sourcing

**All are valid** as long as:
* Once locked, content cannot be changed
* Lock timestamp is recorded
* Version chain is maintained

### Versioning Chain

When a LOCKED asset must be corrected:

```json
{
  "AssetID": "assess-001",
  "AssetType": "Assessment",
  "VersionChain": [
    {
      "Version": 1,
      "CreatedTimestamp": "2026-01-15T10:00:00Z",
      "Content": "Records are not responsive",
      "LockedTimestamp": "2026-01-15T10:01:00Z",
      "LockedBy": "rao@town.example.com",
      "Reason": "Assessment complete"
    },
    {
      "Version": 2,
      "CreatedTimestamp": "2026-01-16T09:30:00Z",
      "Content": "Records ARE responsive (v1 was error)",
      "LockedTimestamp": "2026-01-16T09:31:00Z",
      "LockedBy": "supervisor@town.example.com",
      "Reason": "Correction to assessment",
      "SupersedesVersion": 1
    }
  ]
}
```

Audit trail shows both versions. Decision-maker is identified. Reason is recorded.

---

## Retention and Destruction

### Retention Schedule

| Class | Minimum Period | From | After |
|-------|-----------------|------|-------|
| KEEPER | Indefinite | — | Never |
| REFERENCE | 7 years | CASE Closure | Destroy per policy |
| TRANSACTIONAL | 1-2 years | CASE Closure | Destroy per policy |

**Plus:** Any statute-specific retention (e.g., "payroll records 7 years" per M.G.L. c. 149)

### Destruction Process

Before destroying records:

1. **Verify retention period has passed**
2. **Check for legal hold** (litigation, audit, investigation)
3. **Archive if required** (historical value, state mandate)
4. **Log destruction** (what was destroyed, when, why)
5. **Verify purge** (confirm records are deleted)

**Rule:** Destruction is logged. You can always answer "When were these records destroyed?"

### Legal Hold

If litigation or investigation is pending:

1. Freeze retention clock
2. Mark CASE as "HOLD"
3. Do not destroy anything
4. Resume destruction after hold is released

---

## Compliance Checklist

### For Assets in OPEN CASEs

- [ ] Asset has AssetID (UUID)
- [ ] Asset has AssetType (matches scope)
- [ ] Asset has RetentionClass (KEEPER, REFERENCE, or TRANSACTIONAL)
- [ ] Asset is linked to CaseID
- [ ] Asset modifications are logged

### For Assets in CLOSED CASEs

- [ ] Critical assets are LOCKED
- [ ] LOCKED timestamp is recorded
- [ ] LOCKED reason is documented
- [ ] Version chain is maintained (if corrected)
- [ ] Retention class is set correctly

### For Asset Destruction

- [ ] Retention period has elapsed
- [ ] No legal hold exists
- [ ] Destruction is logged with timestamp and actor
- [ ] Destruction can be audited

---

## Immutability in Different Modules

### VAULTPRR

KEEPER assets:
* Withholding Letter (evidence of exemption decision)
* Approval Letter (evidence of release decision)
* Search Memo (evidence of search)
* Fee Waiver Certificate (evidence of T10 miss)

REFERENCE assets:
* Communication with requester
* Draft analysis
* Counsel memo

---

### VAULT CLERK

KEEPER assets:
* License Approval or Denial
* Inspection Report (if approval decision based on inspection)
* Background Check Result

REFERENCE assets:
* Application
* Supporting documents submitted by applicant

---

### VAULT FISCAL

KEEPER assets:
* Invoice Approval
* Payment Record
* Reconciliation

REFERENCE assets:
* Purchase Order
* Receipt
* Invoice scan

---

## Q&A

**Q: Can I edit a LOCKED asset?**
A: No. If correction is needed, create a new version.

**Q: How long do I keep REFERENCE records?**
A: 7 years from CASE closure, unless statute says longer.

**Q: Can I destroy a KEEPER record?**
A: Rarely. Check statute and legal hold. Usually never.

**Q: What if I find an error in a LOCKED document?**
A: Create a new version with "Correction to v1" reason. Both versions exist in chain.

**Q: Do I have to LOCK everything?**
A: No. Only documents that are evidence of final decisions. Supporting docs can remain mutable.

**Q: Can I LOCK a document before CASE closure?**
A: Ideally not. Lock at closure. Early locking may prevent legitimate corrections.

---

*Immutability is how VAULT creates trustworthy evidence. If you are not clear on this, ask.*
