# VAULT TIME™ — Governance Architecture & Encoding Specification

**Consolidated Version 1.0**  
**Authority:** PublicLogic™  
**Governing Law:** M.G.L. c. 149 (Labor); Local HR Policy

---

## PART I: GOVERNANCE ARCHITECTURE

### Purpose

VAULT TIME™ defines the governed operating system for municipal timesheet entry, approval, and payroll integration.

**Core Principle:** Compensation records must be auditable, immutable, and traceable to preserve employee trust and comply with wage/hour law.

---

### Statutory Authority

| Statute | Authority | Enforcement |
|---|---|---|
| **M.G.L. c. 149** | Fair Labor Standards (minimum wage, overtime) | Overtime rules, wage calculation enforcement |
| **Local HR Policy** | Compensation rules, approval authority | Approval chain, allocation rules |
| **Payroll Schedule** | Payment timing | Payment date enforcement |

### CASE Lifecycle for TIME

```
ENTRY
  ↓ [Employee enters hours]
REVIEW
  ↓ [Manager reviews for accuracy]
APPROVAL
  ↓ [Manager approves]
PAYROLL_POSTING
  ↓ [Posted to payroll system]
CLOSED
```

### Critical Gates

#### Gate 1: Completeness
- Employee entered all required fields (date range, hours, project code, manager)
- Variance from expected hours is explained (if > 10% deviation)

#### Gate 2: Supervisor Approval
- Manager verified accuracy
- Overtime flagged and justified (if applicable)
- Project allocation verified

#### Gate 3: Overtime Authorization
- If hours > 40/week: Finance must approve (budget impact)
- If hours > 48/week: Director approval required

#### Gate 4: Payroll Integration
- Timesheet locked (cannot edit after posting)
- Transferred to payroll system
- Confirmation recorded

### Timers

| Timer | Duration | Enforcement |
|---|---|---|
| **T_ENTRY** | Due by Friday EOD for Fri-Thu week | Reminder sent Wed; hard stop Friday EOD |
| **T_APPROVAL** | Due by Mon 9 AM following week | Manager reminder Thu; auto-escalate if missing |
| **T_PAYROLL** | Posted by Tue 5 PM for Fri paycheck | Hard deadline; no payment processed if not posted |

### Enforcement Rules

```
Rule: Overtime Requires Approval
  IF HoursPerWeek > 40
  AND OvertimeApprovalStatus != "APPROVED":
    BLOCK: Payroll posting blocked
    Action: Route to Finance for approval

Rule: Timesheet is LOCKED After Posting
  IF TimesheetStatus == "POSTED":
    LOCK: No edits permitted
    If correction needed: Create new version (v2)

Rule: No Pay Without Approval
  IF ApprovalStatus != "APPROVED":
    BLOCK: Payroll posting blocked
```

---

## PART II: ENCODING SPECIFICATION

### State Machine

```
States: ENTRY → REVIEW → APPROVAL → PAYROLL_POSTING → CLOSED

Transitions:
  ENTRY → REVIEW: Employee submits
  REVIEW → APPROVAL: Manager reviews (OR requests clarification → back to ENTRY)
  APPROVAL → PAYROLL_POSTING: All gates pass; payroll window open
  PAYROLL_POSTING → CLOSED: Posted to payroll; locked
```

### JSON Schema: Timesheet CASE

```json
{
  "caseId": "time-2026-w06-emp-001",
  "caseType": "TIME",
  "payPeriod": "2026-02-02 to 2026-02-08",
  "employee": {
    "employeeId": "EMP-001",
    "name": "John Smith",
    "department": "Public Works",
    "supervisor": "supervisor@town.example.com"
  },
  "hoursDetails": {
    "regularHours": 40.0,
    "overtimeHours": 0.0,
    "totalHours": 40.0,
    "variance": {
      "percentFromExpected": 0,
      "requiresExplanation": false
    }
  },
  "allocations": [
    {
      "projectCode": "ROADS-001",
      "hours": 25.0,
      "glCode": "6500.101"
    },
    {
      "projectCode": "MAINT-002",
      "hours": 15.0,
      "glCode": "6500.102"
    }
  ],
  "approvalStatus": "PENDING",
  "supervisorApprovalDate": null,
  "financeApprovalDate": null,
  "payrollPostingDate": null,
  "isLocked": false
}
```

### Overtime Approval Matrix

```json
{
  "Hours 0-40": {
    "supervisorApprovalRequired": true,
    "financeApprovalRequired": false,
    "directorApprovalRequired": false,
    "budgetCheck": false
  },
  "Hours 40-48": {
    "supervisorApprovalRequired": true,
    "financeApprovalRequired": true,
    "directorApprovalRequired": false,
    "budgetCheck": true
  },
  "Hours 48+": {
    "supervisorApprovalRequired": true,
    "financeApprovalRequired": true,
    "directorApprovalRequired": true,
    "budgetCheck": true
  }
}
```

### Timer Algorithms

**T_ENTRY: Weekly Timesheet Entry**
```
EffectiveDeadline = Friday 5:00 PM (end of work week)
EnforcementAction:
  IF TimesheetNotSubmitted AND CurrentTime > Deadline:
    1. System locks entry form
    2. Email to employee + supervisor: "Timesheet overdue"
    3. Escalate to HR
    LogEvent: "T_ENTRY missed; hard stop at deadline"
```

**T_APPROVAL: Supervisor Approval**
```
EffectiveDeadline = Monday 9:00 AM following submission
EnforcementAction:
  IF ApprovalNotComplete AND CurrentTime > Deadline:
    1. Reminder email to supervisor
    2. At Tuesday 9 AM: Auto-escalate to manager's manager
    LogEvent: "T_APPROVAL reminder sent; escalation if not approved by Tuesday"
```

**T_PAYROLL: Payroll Posting**
```
EffectiveDeadline = Tuesday 5:00 PM for Friday paycheck
EnforcementAction:
  IF ApprovedTimesheetNotPosted AND CurrentTime > Deadline:
    1. Automatic escalation to payroll processor
    2. Manual posting by payroll team
    LogEvent: "T_PAYROLL exceeded; manual intervention required"
```

### Validation Rules

```
Rule: Employee Cannot Submit for Future Weeks
  IF PayPeriodEndDate > Today + 7 days:
    REJECT timesheet submission
    Error: "Can only submit timesheets for current/past weeks"

Rule: Supervisor Cannot Approve Another's Timesheet
  IF SupervisorApprovingTimesheet
     AND Timesheet.SupervisorId != AuthenticatedActor.Id:
    REJECT approval
    Error: "Only assigned supervisor can approve"

Rule: No Edits After Payroll Posting
  IF TimesheetStatus == "POSTED":
    LOCK: Prevent all edits
    Action: If correction needed, supervisor creates v2
```

### Error Codes

| Code | Condition | Action |
|---|---|---|
| E_OVERTIME_UNAPPROVED | Overtime hours without Finance approval | Block payroll posting |
| E_MISSING_ALLOCATION | Hours entered without project allocation | Reject submission |
| E_SUPERVISOR_MISSING | Timesheet has no assigned supervisor | Escalate to HR |
| E_VARIANCE_UNEXPLAINED | Hours > 10% variance from expected, no explanation | Request explanation from employee |

---

## PART III: Assets and Retention

| Asset | Type | Retention | Locked |
|---|---|---|---|
| TimesheetEntry | REFERENCE | 7 years | After payroll posting |
| SupervisorApproval | KEEPER | 7 years | At approval time |
| PayrollPosting | KEEPER | 7 years | Permanent |
| PaymentConfirmation | TRANSACTIONAL | 2 years | No |

---

*VAULT TIME is the timesheet module. All time records are LOCKED after payroll posting and cannot be edited. Corrections require version chain.*
