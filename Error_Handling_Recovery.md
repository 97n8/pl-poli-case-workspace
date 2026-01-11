# VAULT Error Handling and Recovery

**System Resilience • Failure Modes and Recovery Procedures**

---

## Overview

VAULT systems must handle failures gracefully without losing governance or auditability.

This document defines error categories, handling strategies, and recovery procedures.

---

## Error Categories

### Category A: Non-Recoverable (Stop Immediately)

These errors mean the system cannot continue safely. Stop all processing.

#### A1: Audit Log Corruption

**Description:** Audit entry cannot be written or audit log is compromised.

**Examples:**
* Audit database is offline
* Audit write fails
* Audit log has missing entries (gap in sequence)
* Audit timestamps are out of order

**Handling:**
1. **Stop** — Do not proceed with any CASE transitions
2. **Alert** — Page on-call ops immediately
3. **Log** — Document the error (local file if DB is down)
4. **Restore** — From backup or manual recovery
5. **Resume** — Only after audit integrity is verified

**Recovery Time:** URGENT (< 1 hour)

---

#### A2: LOCKED Asset Mutation

**Description:** LOCKED asset was modified after lock (should be impossible).

**Examples:**
* A LOCKED asset's content changed
* A LOCKED asset was deleted
* Version chain was rewritten

**Handling:**
1. **Stop** — Do not trust data integrity
2. **Investigate** — How did mutation occur?
3. **Alert** — Security review required
4. **Restore** — From backup before mutation
5. **Report** — To PublicLogic and legal

**Recovery Time:** CRITICAL (immediate escalation)

---

#### A3: Timer Computation Failure

**Description:** Deadline calculation is incorrect or produces invalid result.

**Examples:**
* Business day calculation off by 1
* Deadline is in the past (negative days remaining)
* Holiday calendar was corrupted
* Timer computation loop crashes

**Handling:**
1. **Stop** — Do not rely on deadline enforcement
2. **Alert** — Engineering review immediately
3. **Manual override** — Compute deadline by hand
4. **Verify** — Test with known-good dates
5. **Fix** — Correct calculation; re-compute all open CASEs

**Recovery Time:** CRITICAL (< 4 hours)

---

#### A4: Governance Gap (Cannot Proceed)

**Description:** Encoding encounters ambiguity that governance cannot resolve.

**Example:** "When can a PRR be withdrawn?"

**Handling:**
1. **Stop** — Do not guess
2. **Document** — What is ambiguous? Why does it matter?
3. **Escalate** — Send to PublicLogic
4. **Wait** — PublicLogic provides clarification
5. **Resume** — Implement with clarification

**Recovery Time:** 2-3 business days (standard escalation SLA)

---

### Category B: Recoverable (Handle and Continue)

These errors do not affect core governance. Handle and continue.

#### B1: Transient Database Error

**Description:** Database is temporarily unavailable.

**Examples:**
* Connection timeout
* Query timeout (< 30 seconds)
* Lock contention on table

**Handling:**
1. **Retry** — Exponential backoff (1s, 2s, 4s, 8s, 16s)
2. **After 5 retries** — Escalate to B2 (persistent error)
3. **Log** — Error details and retry count
4. **User notice** — "Please try again in a moment"

**Recovery:** Automatic; no manual intervention needed

---

#### B2: Persistent Database Error

**Description:** Database error that does not resolve with retry.

**Examples:**
* Query exceeds timeout repeatedly
* Deadlock between transactions
* Disk space exhausted

**Handling:**
1. **Alert** — Notify ops team
2. **Escalate** — To database administrator
3. **Workaround** — If possible (e.g., read-only queries)
4. **User notice** — "System maintenance; limited functionality"
5. **Restore** — Fix underlying issue; resume normal operation

**Recovery Time:** 1-4 hours (depends on issue)

---

#### B3: Network Connectivity Loss

**Description:** External system (email, payroll, etc.) is unreachable.

**Examples:**
* Email system is down (cannot send PRR response letter)
* Payroll integration fails (cannot post timesheet)
* Records system is offline (cannot query archived records)

**Handling:**
1. **Queue** — Store action (email, posting, etc.) in pending queue
2. **Retry** — Periodically attempt to send (every 5 minutes)
3. **After 24 hours** — Escalate to manual action
4. **Log** — All retry attempts
5. **Alert** — If still pending after 24h

**Recovery:** Automatic when external system recovers

---

#### B4: Malformed Input

**Description:** User provides data that violates schema or constraints.

**Examples:**
* Exemption code that does not exist
* Actor ID that is not valid
* Date string in wrong format
* Amount field is negative (when should be positive)

**Handling:**
1. **Reject** — Return error to user with reason
2. **Log** — Validation failure (who, what, when)
3. **Suggest** — What format is expected?
4. **User fixes** — Resubmit with correct data

**Recovery:** User corrects and resubmits

---

### Category C: Warning (Proceed with Notice)

These errors represent unusual but valid situations. Proceed but flag.

#### C1: Deadline Approaching

**Description:** Deadline is within 2 days and action incomplete.

**Example:** T10 due Friday; today is Wednesday; response not ready.

**Handling:**
1. **Alert** — Warn operator: "T10 deadline in 2 days"
2. **Escalate** — Ask supervisor if extension is needed
3. **Continue** — Process continues; decision is operator's

**Recovery:** Operator acknowledges or takes action

---

#### C2: Unusual Processing Time

**Description:** CASE is taking much longer than typical.

**Example:** PRR has been in GATHERING state for 45 days; typical is 10 days.

**Handling:**
1. **Alert** — "Case is unusually old; check status"
2. **Notify** — RAO or supervisor
3. **Continue** — Process continues
4. **Action** — Operator determines if escalation is needed

**Recovery:** Operator investigates and acts

---

#### C3: Data Quality Issue

**Description:** Data is valid but incomplete or suspicious.

**Examples:**
* Requester email is missing (can we still process?)
* Scope description is blank (is this valid?)
* Redaction spans 100% of document (fully withheld; is this correct?)

**Handling:**
1. **Flag** — Mark CASE with "Data quality" flag
2. **Alert** — Notify operator
3. **Allow override** — Operator can proceed or request clarification
4. **Log** — Flag and operator response

**Recovery:** Operator acknowledges

---

## Error Handling Best Practices

### 1. Always Log Errors

**Rule:** Every error goes to log, regardless of handling.

**Examples:**
```
[2026-02-07T14:32:15Z] ERROR Timer computation
  CaseID: prr-2026-001
  Error: Business day calculation returned negative remaining days
  Details: Receipt date 2026-02-10, due date 2026-02-07
  Handler: CRITICAL - escalated to ops

[2026-02-07T14:35:22Z] WARNING T10 approaching
  CaseID: prr-2026-002
  DaysRemaining: 2
  Status: Response not ready
  Action: RAO notified
```

---

### 2. Never Silently Ignore Errors

**Bad:**
```
try {
  compute_deadline(receipt_date)
} catch (error) {
  // ignore, use default
  deadline = receipt_date + 15  // WRONG
}
```

**Good:**
```
try {
  deadline = compute_deadline(receipt_date)
} catch (error) {
  log("CRITICAL", "Deadline computation failed", {
    case_id: case_id,
    error: error,
    action: "STOP"
  })
  alert_ops()
  throw error  // Re-raise
}
```

---

### 3. Errors Are Audit Events

**Rule:** Every error is logged to audit trail.

**Format:**
```json
{
  "EventID": "evt-error-001",
  "Timestamp": "2026-02-07T14:32:00Z",
  "Action": "Error",
  "ErrorCode": "E_DEADLINE_COMPUTE",
  "ErrorMessage": "Business day calculation failed",
  "Details": {
    "CaseID": "prr-2026-001",
    "Function": "compute_deadline",
    "Input": "2026-02-10",
    "Exception": "ValueError: negative business days"
  },
  "Recovery": "MANUAL - ops intervention required"
}
```

---

### 4. Distinguish Between Errors and Governance Gaps

**Error:** System did something wrong. Fix the system.

**Governance Gap:** Specification is ambiguous. Escalate to PublicLogic.

**Example:**

Bad: "The spec doesn't say what to do, so I'll just delete the records."
→ This is a Governance Gap. Escalate instead.

Good: "The spec says T10 is 10 days. My calculation gave 9 days. Bug in timer computation."
→ This is an Error. Fix the calculation.

---

## Recovery Procedures by Scenario

### Scenario 1: Audit Log Is Down

**Symptom:** Audit write fails; system cannot log changes.

**Procedure:**
1. Stop all CASE state transitions
2. Allow read-only queries (do not change state)
3. Queue pending changes (write to local file if DB is down)
4. Alert: "System in read-only mode; audit recovery in progress"
5. Restore audit database or switch to backup
6. Replay queued changes (with timestamps of when they occurred)
7. Resume normal operation

**Time:** 15-60 minutes depending on cause

---

### Scenario 2: Deadline Computation Is Wrong

**Symptom:** T10 deadline is off by 1 day consistently.

**Procedure:**
1. Identify root cause (holiday not in calendar? Business day logic error?)
2. Stop new deadline computations (mark as manual override)
3. Identify all affected CASEs (query: where deadline_system != deadline_correct)
4. Recalculate deadlines for all CASEs
5. Fix underlying code
6. Recompute all deadlines
7. Resume automated deadline computation
8. Audit: Document all recalculations and reason

**Time:** 2-4 hours

---

### Scenario 3: LOCKED Asset Was Mutated

**Symptom:** LOCKED asset's content changed (should be impossible).

**Procedure:**
1. **STOP all processing immediately**
2. Alert: Security incident
3. Notify legal and PublicLogic
4. Investigate: How did mutation occur? When? What changed?
5. Restore: Revert asset to backup version before mutation
6. Audit: Document mutation, restoration, and investigation
7. Fix: Patch underlying issue (storage layer, permissions, etc.)
8. Test: Verify LOCKED assets cannot be mutated
9. Resume: Only after verification

**Time:** CRITICAL (< 4 hours, may require litigation hold)

---

### Scenario 4: Network Outage (Email Down)

**Symptom:** PRR response letter cannot be sent via email.

**Procedure:**
1. Queue email in pending queue (do not fail)
2. Retry every 5 minutes for 24 hours
3. At 24 hours: Alert RAO, allow manual send via alternative (paper mail, etc.)
4. When email recovers: Automatically send queued items
5. Log all retry attempts

**Time:** Automatic; manual action if > 24h

---

## Escalation Matrix

| Error Type | Severity | Response Time | Action |
|------------|----------|---------------|--------|
| Audit log down | CRITICAL | < 1 hour | Stop, restore, resume |
| LOCKED asset mutated | CRITICAL | < 4 hours | Stop, investigate, restore |
| Timer computation wrong | CRITICAL | < 4 hours | Fix, recompute all, resume |
| Governance gap | HIGH | 2-3 business days | Document, escalate to PublicLogic |
| Persistent DB error | HIGH | 1-4 hours | Alert ops, escalate, workaround |
| Network outage | MEDIUM | Automatic recovery | Queue, retry, manual if > 24h |
| Deadline approaching | MEDIUM | 2 days notice | Alert operator |
| Data quality issue | LOW | Next business day | Flag, notify operator |

---

## Communication

### To Operators

When error occurs:

```
⚠️ WARNING: T10 deadline approaching for PRR-2026-001
  Deadline: Friday, Feb 7
  Days remaining: 2
  Status: Response draft not ready
  
  ACTION: Confirm response will be delivered by deadline,
          or request extension from Supervisor
```

### To Auditors

When critical error occurs:

```
🔴 CRITICAL ERROR LOG

Date: 2026-02-07T14:32:00Z
Error: Timer computation failure
CaseID: prr-2026-001
Details: Business day calculation returned invalid result
Cause: [TBD after investigation]
Recovery: [Manual deadline computed as 2026-02-21]
Resolution: [Code fix deployed; all CASEs recalculated]
Audit trail: All actions logged with timestamps
```

### To PublicLogic

When governance gap or system failure:

```
ESCALATION: Governance Gap

Case: Cannot process PRR withdrawal after records gathered
Issue: Spec does not address this scenario
Impact: Blocking 1 CASE; waiting for guidance
Requested clarification: Is withdrawal allowed? What happens to gathered records?
Response timeline: 2-3 business days preferred
```

---

## Testing Error Scenarios

Before deployment, test:

- [ ] Audit log failure (and recovery)
- [ ] Timer computation with invalid input
- [ ] Network outage while writing asset
- [ ] Concurrent access causing lock
- [ ] Large input causing timeout
- [ ] Missing required field in CASE
- [ ] Invalid state transition attempt
- [ ] LOCKED asset mutation attempt (should fail)

All tests should pass with proper error handling and logging.

---

*Errors happen. Governance requires that they are handled safely, logged completely, and escalated appropriately. Do not hide errors. Make them visible and recoverable.*
