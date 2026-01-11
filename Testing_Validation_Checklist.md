# VAULT Testing and Validation Checklist

**Implementation Verification • All Modules**

---

## Overview

Before deployment, verify VAULT implementation against these test scenarios.

Each scenario tests a specific governance requirement. All must pass.

---

## CASE and Entity Tests

### Test 1: CASE Creation and Identity

**Requirement:** Every CASE gets immutable ID; cannot be changed.

**Test:**
```
1. Create CASE with random UUID
2. Read CASE back
3. Attempt to update CaseID
4. Verify: UPDATE fails; CaseID unchanged; audit log shows rejection
```

**Pass Criteria:** CaseID is immutable; attempt to change is rejected and logged

**Severity:** CRITICAL

---

### Test 2: CASE Lifecycle States

**Requirement:** CASE can only transition forward; specific gates guard transitions.

**Test:**
```
1. Create CASE (state = INTAKE)
2. Transition to ASSESSMENT (valid)
3. Transition to GATHERING (depends on gates passing)
4. Attempt to transition back to INTAKE (invalid)
5. Verify: Backward transition rejected; audit log shows rejection reason
```

**Pass Criteria:** Only forward transitions allowed; gates enforced; reversions blocked

**Severity:** CRITICAL

---

### Test 3: Scope Versioning

**Requirement:** If scope changes, new version created (not overwrite).

**Test:**
```
1. Create CASE with Scope v1: "Records about ZBA meeting Jan 15"
2. Requester clarifies: "Only board correspondence, not staff notes"
3. Update scope to v2
4. Verify: v1 still exists; v2 linked; both timestamped; reason recorded
```

**Pass Criteria:** Version chain intact; both versions readable; history preserved

**Severity:** HIGH

---

## Timer and Deadline Tests

### Test 4: Business Day Calculation (Standard Week)

**Requirement:** T10 = 10 business days from effective receipt.

**Test:**
```
Receipt: Monday, Jan 20, 2025 (normal weekday)
Add 10 business days:
  Day 1: Mon 1/20
  Day 2: Tue 1/21
  Day 3: Wed 1/22
  Day 4: Thu 1/23
  Day 5: Fri 1/24
  Day 6: Mon 1/27 (weekend skip)
  Day 7: Tue 1/28
  Day 8: Wed 1/29
  Day 9: Thu 1/30
  Day 10: Fri 1/31

Expected T10 due date: Friday, Jan 31, 2025

Verify: System computes exactly Fri 1/31
```

**Pass Criteria:** Computed date matches expected date

**Severity:** CRITICAL

---

### Test 5: Weekend Shift

**Requirement:** If received on Saturday, shift to Monday.

**Test:**
```
Receipt (actual): Saturday, Jan 25, 2025
Expected EffectiveReceiptDate: Monday, Jan 27, 2025

T10 deadline: 10 business days from Mon 1/27 = Friday, Feb 7

Verify: System computes Mon 1/27 as start; Fri 2/7 as due date
```

**Pass Criteria:** Non-business-day shifts to next business day

**Severity:** CRITICAL

---

### Test 6: Holiday Handling

**Requirement:** Legal holidays are non-business days.

**Test:**
```
Receipt: Friday, Feb 14, 2026
Presidents' Day: Monday, Feb 16, 2026

Add 10 business days:
  Day 1-5: Fri 2/14 through Tue 2/17 (skip weekend + Presidents' Day)
  Day 6-10: Wed 2/18 through Mon 2/23

Expected T10: Monday, Feb 23, 2026

Verify: System skips Presidents' Day; computes Mon 2/23
```

**Pass Criteria:** Holidays correctly excluded

**Severity:** CRITICAL

---

### Test 7: Municipal Closure Handling

**Requirement:** Municipal-specific closures are non-business days.

**Test:**
```
Configuration: Town meeting day = May 1, 2026 (closure)

Receipt: April 27, 2026 (Monday)
Add 10 business days:
  Day 1-5: Mon 4/27 through Fri 5/1 (but 5/1 is closure, skip to 5/4)
  Day 6-10: Mon 5/4 through Fri 5/8

Expected T10: Friday, May 8, 2026

Verify: System respects municipal closure configuration
```

**Pass Criteria:** Municipal closures excluded from count

**Severity:** HIGH

---

### Test 8: Tolling Clock Pause and Resume

**Requirement:** When tolled, deadline clock pauses; days add back when resumed.

**Test:**
```
Original T10 due: Friday, Feb 7, 2026

Jan 29: Clarification requested (TOLL START)
Feb 3: Clarification received (TOLL END)
Tolled days: 5 business days (29, 30, 31, 2, 3)

New T10 due: Friday, Feb 7 + 5 days = Friday, Feb 12, 2026

Verify: 
  - Toll event logged (start, end, reason, days)
  - New deadline computed
  - Audit trail shows tolling
```

**Pass Criteria:** Tolled days add back; deadline extends correctly

**Severity:** HIGH

---

### Test 9: Deadline Miss Detection

**Requirement:** If deadline passes without action, enforcement triggers automatically.

**Test (T10 Fee Waiver):**
```
T10 due date: Friday, Feb 7, 2026
Actual response sent: Tuesday, Feb 11, 2026

System check on Feb 11:
  - T10 due date was Feb 7
  - Response sent Feb 11 (4 days late)
  - Enforce rule: FeesAllowed = false

Verify:
  - FeesAllowed set to false
  - Enforcement logged
  - Cannot override (FeesAllowed remains false)
```

**Pass Criteria:** Deadline miss triggers enforcement automatically; no discretion

**Severity:** CRITICAL

---

### Test 10: T25 Forecast Gate

**Requirement:** If completion forecast > T25, extension required before proceeding.

**Test:**
```
T25 due: Thursday, Feb 19, 2026

Forecast: "Records will be gathered by March 5" (14 days past T25)

Gate evaluation:
  - Forecast completion > T25
  - Require: Requester agreement OR Supervisor petition
  
Without agreement/petition:
  - Transition to GATHERING blocked
  - TransitionBlockers = ["Extension required"]

With agreement received:
  - TransitionBlockers cleared
  - Transition to GATHERING allowed
  - New deadline computed (T25 + extension period)

Verify: Gate blocks until agreement/petition
```

**Pass Criteria:** T25 gate enforces extension requirement

**Severity:** CRITICAL

---

### Test 11: T90 Appeal Window (Post-Closure)

**Requirement:** After closure, T90 timer activates (90 calendar days, not business days).

**Test:**
```
CASE closed: Friday, March 7, 2026

T90 window: 90 calendar days from March 7 = Friday, June 5, 2026

Appeal attempt on June 4: ALLOWED (within window)
Appeal attempt on June 6: REJECTED (T90 expired)

Verify:
  - T90 timer only activates at closure
  - 90 calendar days (not business days)
  - Rejects appeals after expiration
```

**Pass Criteria:** T90 calculated correctly; enforces appeal deadline

**Severity:** HIGH

---

## Asset and Immutability Tests

### Test 12: Asset Lock Mechanism

**Requirement:** Once LOCKED, asset cannot be edited.

**Test:**
```
1. Create asset: WithholdingLetter
   Content: "Records withheld per exemption §26b (trade secret)"
   Locked: false

2. Lock asset
   Locked: true
   LockedTimestamp: 2026-02-07T16:45:00Z
   LockedBy: rao@town.example.com

3. Attempt to edit content
4. Verify: UPDATE rejected; audit log shows rejection
```

**Pass Criteria:** LOCKED assets cannot be modified; attempt is rejected and logged

**Severity:** CRITICAL

---

### Test 13: Version Chain for Corrections

**Requirement:** If LOCKED asset needs correction, new version created (not edit).

**Test:**
```
1. Asset v1: ASSESSMENT_DECISION
   Content: "Records not responsive"
   Locked: true

2. Error found: "Actually records ARE responsive"

3. Create Asset v2: ASSESSMENT_DECISION
   Content: "Records ARE responsive"
   Locked: true
   SupersedesVersion: 1

4. Link chain:
   - Asset v1 exists (original)
   - Asset v2 exists (correction)
   - Both locked
   - Audit trail shows both

Verify: Both versions exist; chain is clear; audit trail complete
```

**Pass Criteria:** Corrections create new versions; no rewriting history

**Severity:** CRITICAL

---

### Test 14: Retention Class Enforcement

**Requirement:** Assets are classified and retained per class.

**Test:**
```
Asset 1: WithholdingLetter
  RetentionClass: KEEPER
  Expected: Permanent

Asset 2: SearchMemo
  RetentionClass: REFERENCE
  Expected: 7 years from closure

Asset 3: ProcessingNote
  RetentionClass: TRANSACTIONAL
  Expected: 1-2 years from closure

Verify:
  - Each asset has retention class
  - System enforces retention (prevents premature deletion)
  - Audit tracks destruction
```

**Pass Criteria:** All assets classified; retention rules enforced

**Severity:** HIGH

---

## Audit and Logging Tests

### Test 15: Audit Log Completeness

**Requirement:** Every action creates audit entry.

**Test:**
```
Sequence:
  1. Create CASE
  2. Update scope
  3. Create asset
  4. Lock asset
  5. Transition state
  6. Enforce rule

Audit log:
  ✓ Entry 1: Case creation
  ✓ Entry 2: Scope update
  ✓ Entry 3: Asset creation
  ✓ Entry 4: Asset lock
  ✓ Entry 5: State transition
  ✓ Entry 6: Rule enforcement

Verify: All 6 entries present; no gaps; correct order
```

**Pass Criteria:** Audit log is complete; every action logged

**Severity:** CRITICAL

---

### Test 16: Audit Log Immutability

**Requirement:** Audit entries cannot be edited, deleted, or reordered.

**Test:**
```
1. Create audit entry: EventID = evt-001, timestamp = 2026-02-07T14:32:00Z
2. Attempt to UPDATE entry: Change timestamp to earlier time
3. Verify: UPDATE fails; entry unchanged
4. Attempt to DELETE entry
5. Verify: DELETE fails; entry remains
6. Attempt to INSERT entry out of order
7. Verify: INSERT fails; order intact
```

**Pass Criteria:** Audit log is append-only; no mutations; no reordering

**Severity:** CRITICAL

---

### Test 17: Actor Identity Logging

**Requirement:** Every action has actor; no anonymous changes.

**Test:**
```
Actor 1: RAO (rao@town.example.com)
  - Creates CASE
  - Audit log shows: Actor = rao@town.example.com
  
Actor 2: Supervisor (supervisor@town.example.com)
  - Approves extension
  - Audit log shows: Actor = supervisor@town.example.com

Attempt: System action with no actor
Verify: System rejects; requires actor identity
```

**Pass Criteria:** All actions attributed; no anonymous; actor queryable

**Severity:** HIGH

---

## Gate and Enforcement Tests

### Test 18: Gate Blocks Invalid Transition

**Requirement:** Gates prevent transition if conditions not met.

**Test:**
```
Gate: T25 Forecast
Condition: If forecast completion > T25 AND no agreement, block

Scenario 1: Forecast completion ≤ T25
  → Gate passes → Transition allowed ✓

Scenario 2: Forecast > T25, requester agrees
  → Gate passes → Transition allowed ✓

Scenario 3: Forecast > T25, no agreement, no petition
  → Gate FAILS → Transition BLOCKED ✓
  → TransitionBlockers = ["Extension required"]
  → Audit entry: "T25 gate failed"

Verify: Gate correctly blocks/allows
```

**Pass Criteria:** Gates enforce conditions; block when required

**Severity:** CRITICAL

---

### Test 19: Enforcement Without Discretion

**Requirement:** When deadline missed, consequence executes automatically.

**Test (T10 Miss):**
```
T10 due: Friday, Feb 7
Response sent: Tuesday, Feb 11

Scenario 1: RAO wants to charge fees anyway
  → System rejects (FeesAllowed = false, locked)
  → Audit log: "Fee override attempted; rejected"

Scenario 2: System automatically waives fees
  → FeesAllowed = false
  → Invoice generation skipped
  → Audit log: "T10 missed; fees waived automatically"

Verify: Consequence is automatic; no discretion
```

**Pass Criteria:** Enforcement is mandatory; no override possible

**Severity:** CRITICAL

---

## Record Classification Tests

### Test 20: Correct Asset Classification

**Requirement:** Assets are classified as KEEPER, REFERENCE, or TRANSACTIONAL.

**Test (PRR Module):**
```
Asset 1: WithholdingLetter
  Classification: KEEPER (evidence of decision)
  ✓ Correct

Asset 2: SearchMemo
  Classification: REFERENCE (supporting, not decision itself)
  ✓ Correct

Asset 3: StatusNote ("Page 3 scanned")
  Classification: TRANSACTIONAL (temporary working note)
  ✓ Correct

Verify: Each asset correctly classified per rules
```

**Pass Criteria:** Classification follows governance rules

**Severity:** HIGH

---

## Module-Specific Tests

### Test 21: PRR — Exemption Application

**Requirement:** Exemptions are documented; segregability is applied.

**Test:**
```
Document: Board meeting minutes (20 pages)

Exemption scan:
  - Pages 1-5: No exempt information
  - Pages 6-10: Personnel discussions (exempt §26a)
  - Pages 11-20: Policy decisions (not exempt)

Segregability:
  - Release pages 1-5 (no exempt)
  - Redact pages 6-10 (exempt)
  - Release pages 11-20 (not exempt)

Verify:
  - Exemption tags applied correctly
  - Segregability respected
  - Redacted version generated
  - Audit trail shows all decisions
```

**Pass Criteria:** Exemptions applied; segregability enforced

**Severity:** HIGH (PRR-specific)

---

### Test 22: FISCAL — 3-Way Match

**Requirement:** Invoice, PO, and receipt amounts match before payment.

**Test:**
```
Invoice: $4,500
PO: $4,500
Receipt: $4,500
→ Match → Payment allowed ✓

Invoice: $4,500
PO: $4,000
Receipt: $4,500
→ Variance → Require approval before payment ✓

Verify: 3-way match enforced; variance blocks payment
```

**Pass Criteria:** 3-way match verified; variance detected

**Severity:** HIGH (FISCAL-specific)

---

### Test 23: TIME — Supervisor Approval

**Requirement:** Timesheet requires supervisor approval before payroll posting.

**Test:**
```
Employee enters: 40 hours
Supervisor reviews: APPROVE
Status: APPROVED

Payroll job runs: Pulls APPROVED timesheets only

Verify: Non-approved timesheets excluded from payroll
```

**Pass Criteria:** Approval gate enforced; payroll respects status

**Severity:** HIGH (TIME-specific)

---

## Cross-System Tests

### Test 24: Audit Trail Across Modules

**Requirement:** Each module logs independently; audit trails are self-contained.

**Test:**
```
Employee timeline:
  1. ONBOARD CASE created (Feb 1)
     - Audit log: "Onboarding started"
  
  2. TIME CASE created (Feb 2, timesheet)
     - Audit log: "Timesheet entry"
  
  3. FISCAL CASE created (Feb 7, invoice)
     - Audit log: "Invoice received"

Verify:
  - Each CASE has independent audit log
  - No cross-contamination
  - Employee identity ties cases together (business, not technical)
```

**Pass Criteria:** Audit logs are independent; no bleeding

**Severity:** MEDIUM

---

## Scalability Tests

### Test 25: Large Audit Log Performance

**Requirement:** System handles high-volume logging without degradation.

**Test:**
```
Create 100 CASEs
Generate 50 events each (5,000 total events)

Measure:
  - Query time for "all events by actor"
  - Query time for "all events by date range"
  - Query time for "all events for CASE"

Verify: Queries complete in < 1 second
```

**Pass Criteria:** Audit queries are performant

**Severity:** MEDIUM

---

### Test 26: Concurrent State Transitions

**Requirement:** Multiple simultaneous transitions don't corrupt state.

**Test:**
```
Setup: Two operators accessing same CASE simultaneously
  Operator A: Transitions from ASSESSMENT to GATHERING
  Operator B: Transitions from ASSESSMENT to GATHERING

Result:
  - One transition succeeds
  - One transition fails (CASE already moved)
  - Audit log shows both attempts
  - Final state is consistent (GATHERING)

Verify: No race conditions; state is consistent
```

**Pass Criteria:** Concurrent access handled safely

**Severity:** MEDIUM

---

## Deployment Readiness Checklist

Before go-live:

- [ ] All CRITICAL tests pass (Tests 1-3, 4-9, 11, 12-13, 15-19)
- [ ] All HIGH tests pass (Tests 5-6, 8, 10, 14, 20-23)
- [ ] All MEDIUM tests pass (Tests 24-26)
- [ ] Deadline computation verified with 50+ dates
- [ ] Audit log tested with 10K+ events
- [ ] Asset locking tested in production DB
- [ ] Escalation path tested (can contact PublicLogic within SLA)
- [ ] Documentation reviewed (README, charters, glossary)
- [ ] Training material tested with end users

---

## Test Automation

These tests should be:
* **Unit tests** — Timer computation, gate logic
* **Integration tests** — CASE creation through closure
* **Audit tests** — Immutability, completeness
* **Load tests** — Scalability with 10K+ CASEs

Recommend: Test harness in CI/CD pipeline. All tests run on every commit.

---

*All tests must pass before go-live. Do not deploy if any test fails.*
