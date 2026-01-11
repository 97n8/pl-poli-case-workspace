# VAULT FIX™ — Encoding Specification

**Implementation Contract • Version 1.0**  
**Authority:** PublicLogic™

---

## Part I: State Machine

```
States: INTAKE → TRIAGE → SCHEDULING → IN_PROGRESS → COMPLETION_REVIEW → CLOSURE

Transitions:
  INTAKE → TRIAGE: Priority assigned
  TRIAGE → SCHEDULING: Skill determined; assignment decision made
  SCHEDULING → IN_PROGRESS: Scheduled date reached; work starts
  IN_PROGRESS → COMPLETION_REVIEW: Work claimed complete
  COMPLETION_REVIEW → CLOSURE: Verification passed; work locked
  COMPLETION_REVIEW → IN_PROGRESS: Verification failed; return to work
```

---

## Part II: JSON Schema (Work Order CASE)

```json
{
  "caseId": "fix-2026-001",
  "caseType": "FIX",
  "requestType": "WorkOrder",
  "workScope": "Roof leak in gymnasium",
  "priority": "HIGH",
  "slaResponse": "4 hours",
  "slaCompletion": "5 business days",
  "requestDate": "2026-01-15T09:30:00Z",
  "requester": {
    "name": "Facilities Director",
    "email": "facilities@town.example.com"
  },
  "asset": {
    "building": "High School",
    "system": "Roof",
    "location": "Gymnasium"
  },
  "assignment": {
    "skillRequired": "Roofing",
    "assignedTo": "Licensed Roofer ABC",
    "assignmentType": "Contractor",
    "scheduledDate": "2026-01-18"
  },
  "completion": {
    "actualStartDate": "2026-01-18",
    "completionDate": "2026-01-19",
    "verificationStatus": "VERIFIED",
    "verifiedBy": "Building Inspector",
    "verificationMethod": "Inspection + sign-off"
  },
  "costs": {
    "estimatedCost": 2500.00,
    "actualCost": 2350.00,
    "glCode": "6400.001"
  }
}
```

---

## Part III: Priority SLA Timers

### T_RESPONSE_SLA (SLA Response Time)

```
Priority CRITICAL: 2 hours
Priority HIGH: 4 hours
Priority NORMAL: 24 hours
Priority LOW: 5 business days

IF CurrentTime > T_RESPONSE_SLA AND AssignmentStatus = UNASSIGNED:
  Log: "Response SLA missed; escalate"
  Action: Escalate to supervisor; flag case
```

### T_COMPLETION_SLA (SLA Completion Time)

```
Priority CRITICAL: 24 hours from response
Priority HIGH: 5 business days
Priority NORMAL: 14 business days
Priority LOW: 30 business days

IF CurrentTime > T_COMPLETION_SLA AND CompletionStatus != VERIFIED:
  Log: "Completion SLA missed; escalate"
  Action: Notify requester; flag case
```

---

## Part IV: Validation Rules

### Rule: Skill Must Be Assigned

```
IF SkillRequired = null:
  REJECT: Cannot schedule without skill identification
  Action: Return to TRIAGE; identify required skill
```

### Rule: Verification Required Before Closure

```
IF VerificationStatus != VERIFIED:
  BLOCK: Cannot transition to CLOSURE
  Action: Return to COMPLETION_REVIEW; complete verification
```

### Rule: Completion Verification is LOCKED

```
IF VerificationRecorded:
  LOCK: Cannot be edited
  IF edit attempted:
    BLOCK: Immutable
    Action: Create new verification version if correction needed
```

---

## Part V: Error Codes

| Code | Condition | Action |
|---|---|---|
| E_PRIORITY_MISSING | Priority not assigned | Assign priority; set SLA |
| E_SKILL_MISSING | Skill required not identified | Return to TRIAGE; identify skill |
| E_SLA_RESPONSE_MISSED | Response SLA exceeded | Escalate; notify supervisor |
| E_SLA_COMPLETION_MISSED | Completion SLA exceeded | Escalate; notify requester |
| E_VERIFICATION_FAILED | Work incomplete; verification failed | Return to IN_PROGRESS |
| E_CANNOT_EDIT_LOCKED | Edit locked verification | Explain immutability |

---

## Part VI: Deployment Checklist

- [ ] Priority classification and SLA times configured
- [ ] Response SLA timers tested for all priorities
- [ ] Completion SLA timers tested for all priorities
- [ ] Assignment logic tested (in-house vs. contractor)
- [ ] Verification workflow tested
- [ ] Cost tracking tested
- [ ] GL posting integration tested
- [ ] Escalation alerts tested
- [ ] Verification signatures locked and audit logged

---

*VAULT FIX Encoding Specification v1.0.*
