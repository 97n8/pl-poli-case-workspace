# VAULT ONBOARD™ — Governance Architecture & Encoding Specification

**Consolidated Version 1.0**  
**Authority:** PublicLogic™  
**Governing Law:** M.G.L. c. 149 (Labor); Local HR Policy

---

## PART I: GOVERNANCE ARCHITECTURE

### Purpose

VAULT ONBOARD™ defines the governed operating system for municipal employee onboarding.

**Core Principle:** Onboarding must be complete, documented, and transferable so that no employee is dependent on any individual's memory or tribal knowledge.

---

### CASE Lifecycle for Onboarding

```
INTAKE
  ↓ [Offer accepted; onboarding case created]
PRE_START
  ↓ [Background check ordered; IT setup scheduled]
DAY_ONE
  ↓ [Employee arrives; welcome & orientation]
SYSTEMS_SETUP
  ↓ [IT account created; access provisioned]
TRAINING
  ↓ [Policy training, department training]
READINESS_REVIEW
  ↓ [Manager confirms employee ready to work independently]
CLOSED
  ↓ [Onboarding complete; employee fully productive]
```

### Critical Tasks (Non-Negotiable)

Every onboarding must complete:

1. **Background Check** — Ordered before start; must clear before first day
2. **I-9 Documentation** — Completed day one
3. **IT Account Creation** — Email, network access, systems
4. **Badge/Access Card** — Building access provisioned
5. **Policy Training** — HR handbook, safety, compliance
6. **Department Orientation** — Role-specific training
7. **Manager Sign-Off** — Confirms readiness to work independently

**Rule:** No task can be skipped. All must be documented and locked.

---

### Task Dependencies

```
Offer Accepted
  ↓
Background Check (parallel: IT setup, workspace prep)
  ├─ Background clear? (must be YES before Day 1)
  └─ If NO: Do not proceed to Day 1; escalate to HR
  ↓
Day 1: Welcome + I-9 + Badge
  ↓
Systems Access (email, network, databases)
  ├─ Email working? (prerequisite for training materials)
  ├─ Network access? (prerequisite for online training)
  └─ All systems tested
  ↓
Policy Training (HR, safety, compliance)
  ├─ HR handbook completed
  ├─ Safety briefing completed
  └─ Compliance training completed
  ↓
Department Training (role-specific)
  ├─ Role documentation provided
  ├─ Key processes explained
  └─ Mentor/buddy assigned
  ↓
Proficiency Verification
  ├─ Manager confirms employee can perform core tasks
  ├─ Manager has met with employee at least 3 times
  └─ No concerns identified
  ↓
Manager Sign-Off → CLOSED
```

---

### Governance Rules

**Rule 1: Background Check is Mandatory**
- Cannot proceed to Day 1 if background check not cleared
- Escalate to HR if background check reveals issues

**Rule 2: All Training Tasks Must Be Documented**
- Training completion locked as immutable record
- Cannot delete or edit training records once locked

**Rule 3: Manager Sign-Off is Final Gate**
- Manager must explicitly confirm readiness
- Cannot close CASE without sign-off
- Sign-off is locked (immutable)

**Rule 4: No Employee Starts Without Complete Onboarding**
- System blocks payroll start date until onboarding closed
- No exceptions without executive approval (logged)

---

## PART II: ENCODING SPECIFICATION

### State Machine

```
States: INTAKE → PRE_START → DAY_ONE → SYSTEMS_SETUP → TRAINING → READINESS_REVIEW → CLOSED

Transitions:
  INTAKE → PRE_START: Offer accepted
  PRE_START → DAY_ONE: Background check cleared; start date reached
  DAY_ONE → SYSTEMS_SETUP: I-9 and badge completed
  SYSTEMS_SETUP → TRAINING: All IT systems confirmed working
  TRAINING → READINESS_REVIEW: Training completed
  READINESS_REVIEW → CLOSED: Manager sign-off received
```

### JSON Schema (Onboarding CASE)

```json
{
  "caseId": "onboard-2026-emp-001",
  "caseType": "ONBOARD",
  "employee": {
    "employeeId": "EMP-2026-001",
    "name": "Jane Doe",
    "department": "Finance",
    "position": "Accountant",
    "startDate": "2026-02-01",
    "salary": 65000.00,
    "supervisor": "CFO"
  },
  "backgroundCheck": {
    "orderedDate": "2026-01-10",
    "completedDate": "2026-01-20",
    "result": "CLEAR",
    "status": "APPROVED"
  },
  "preStart": {
    "offerAcceptedDate": "2026-01-08",
    "workspaceAssigned": "Room 212",
    "hardwareOrdered": true,
    "itAccountScheduled": "2026-01-28"
  },
  "dayOne": {
    "actualArrivalDate": "2026-02-01T08:30:00Z",
    "i9Completed": true,
    "badgeIssued": true,
    "welcomePacketProvided": true,
    "departmentIntroduction": true
  },
  "systemsSetup": {
    "emailCreated": true,
    "networkAccessGranted": true,
    "payrollSystemAccess": true,
    "allSystemsTestConfirmed": "2026-02-02"
  },
  "training": {
    "hrHandbookCompleted": { "date": "2026-02-02", "locked": true },
    "safetyBriefingCompleted": { "date": "2026-02-02", "locked": true },
    "complianceTrainingCompleted": { "date": "2026-02-03", "locked": true },
    "departmentTrainingCompleted": { "date": "2026-02-05", "locked": true }
  },
  "readinessReview": {
    "managerMeetingsDates": ["2026-02-02", "2026-02-03", "2026-02-05"],
    "managerConfirmsReadiness": true,
    "managerSignOffDate": "2026-02-05T17:00:00Z",
    "managerSignOffLocked": true
  }
}
```

### Task Checklist Schema

```json
{
  "prStartTasks": [
    { "task": "Background check ordered", "status": "COMPLETE", "dueDate": "hire_date - 10 days" },
    { "task": "IT account creation scheduled", "status": "COMPLETE", "dueDate": "hire_date - 2 days" },
    { "task": "Workspace assigned and prepared", "status": "COMPLETE", "dueDate": "hire_date - 5 days" },
    { "task": "Hardware ordered", "status": "COMPLETE", "dueDate": "hire_date - 7 days" }
  ],
  "dayOneTasks": [
    { "task": "Welcome & orientation", "status": "COMPLETE", "dueDate": "start_date" },
    { "task": "I-9 completion", "status": "COMPLETE", "dueDate": "start_date" },
    { "task": "Badge/access card issued", "status": "COMPLETE", "dueDate": "start_date" },
    { "task": "Department introduction", "status": "COMPLETE", "dueDate": "start_date" }
  ],
  "systemsSetupTasks": [
    { "task": "Email account working", "status": "COMPLETE", "dueDate": "start_date + 1" },
    { "task": "Network access verified", "status": "COMPLETE", "dueDate": "start_date + 1" },
    { "task": "Payroll system access", "status": "COMPLETE", "dueDate": "start_date + 1" }
  ],
  "trainingTasks": [
    { "task": "HR handbook training", "status": "COMPLETE", "locked": true, "dueDate": "start_date + 1" },
    { "task": "Safety briefing", "status": "COMPLETE", "locked": true, "dueDate": "start_date + 1" },
    { "task": "Compliance training", "status": "COMPLETE", "locked": true, "dueDate": "start_date + 2" },
    { "task": "Department training", "status": "COMPLETE", "locked": true, "dueDate": "start_date + 5" }
  ]
}
```

### Validation Rules

```
Rule: Background Check Must Clear
  IF BackgroundCheckResult != CLEAR:
    BLOCK: Cannot proceed to DAY_ONE
    Action: Escalate to HR; do not start employee

Rule: All Training Must Be Locked
  IF TrainingStatus != LOCKED:
    BLOCK: Cannot proceed to READINESS_REVIEW
    Action: Complete all training first

Rule: Manager Sign-Off is LOCKED
  IF ManagerSignOffRecorded:
    LOCK: Cannot be edited
    IF edit attempted:
      BLOCK: Immutable
```

### Error Codes

| Code | Condition | Action |
|---|---|---|
| E_BACKGROUND_NOT_CLEARED | Background check failed/pending | Escalate to HR; block start |
| E_BACKGROUND_MISSING | No background check on file | Order immediately |
| E_I9_NOT_COMPLETED | I-9 not completed by day one | Escalate to HR; legal risk |
| E_TRAINING_INCOMPLETE | Training not completed | Block readiness review |
| E_MANAGER_SIGNOFF_MISSING | Manager has not signed off | Escalate to supervisor |
| E_CANNOT_EDIT_LOCKED | Edit locked training record | Explain immutability |

### Timers (Soft — Targets, Not Enforcement)

```
T_BACKGROUND_CHECK: 10 business days before start date
  Target: Background check completed and cleared

T_SYSTEMS_SETUP: Start date + 1 business day
  Target: All IT systems working by day 2

T_TRAINING: Start date + 5 business days
  Target: All training completed by end of first week

T_READINESS: Start date + 10 business days
  Target: Manager sign-off by end of second week

Note: These are targets/SLAs, not hard enforcement like PRR.
Escalation only if significantly delayed (>1 week past target).
```

---

## PART III: Assets and Retention

| Asset | Type | Retention | Locked |
|---|---|---|---|
| OfferLetter | KEEPER | Permanent | Yes |
| BackgroundCheckResult | KEEPER | 7 years minimum | Yes |
| I9Document | KEEPER | 3 years (USCIS requirement) | Yes |
| TrainingRecords | KEEPER | 7 years | Yes (after completion) |
| ManagerSignOff | KEEPER | Permanent | Yes |

---

## PART IV: Deployment Checklist

- [ ] Background check integration configured
- [ ] Task checklist system operational
- [ ] IT account creation process documented
- [ ] Training materials prepared and versioned
- [ ] Manager sign-off workflow tested
- [ ] All training records locked automatically after completion
- [ ] Payroll system blocks start if onboarding not closed
- [ ] All error codes tested
- [ ] Retention (7 years minimum) configured

---

*VAULT ONBOARD is the simplest VAULT module. No deadlines enforced. All tasks must be completed; all records locked. Manager sign-off is final gate.*
