# VAULT TIME Module — Charter

**Timesheet Entry, Approval, and Compensation • HR/Payroll**

---

## Module Purpose

VAULT TIME enforces timesheet and compensation processes as CASE-based work.

Each pay period and employee is one CASE (or sub-CASEs per week).

---

## Statutory Authority

**M.G.L. c. 149** — Labor Laws (minimum wage, overtime, etc.)
**Local Payroll Policies**

---

## CASE Model for TIME (Weekly Timesheet)

### Identity

```json
{
  "CaseID": "time-2026-w06-emp-001",
  "CaseType": "TIME",
  "PayPeriod": "2026-02-02 to 2026-02-08",
  "EmployeeID": "EMP-001"
}
```

### Subject

```json
{
  "Employee": {
    "EmployeeID": "EMP-001",
    "Name": "John Smith",
    "Role": "Maintenance Worker",
    "Department": "Public Works"
  },
  "PayPeriod": "Week of Feb 2, 2026",
  "HoursExpected": 40,
  "RateCategory": "Hourly"
}
```

### Assets

```json
{
  "Assets": [
    {"AssetType": "TimesheetEntry", "RetentionClass": "KEEPER"},
    {"AssetType": "ProjectAllocation", "RetentionClass": "KEEPER"},
    {"AssetType": "SupervisorApproval", "RetentionClass": "KEEPER", "Locked": true},
    {"AssetType": "PayrollPosting", "RetentionClass": "REFERENCE"}
  ]
}
```

---

## Module-Specific Rules

### Entry and Verification

Employee enters hours. Supervisor verifies:
* Hours are accurate
* Allocations are correct
* Overtime is properly authorized

### Correction Protocol

If error found after approval:
* Create new timesheet version
* Record reason for change
* Audit trail shows both versions
* Both locked after payroll runs

### Payroll Integration

After CASE closure:
* Data passed to payroll system
* Payroll calculates pay
* Compensation recorded

---

## Process Flow (High Level)

```
Entry → Verification → Allocation → Approval → Payroll Posting → Closure
```

---

## Decision Tables

### Gate 1: Supervisor Approval

| Hours Entered | Variance from Expected | Requires Explanation? | Action |
|---|---|---|---|
| 40 | 0 | No | Approve quickly |
| 38 | -2 | No (within tolerance) | Approve |
| 32 | -8 | Yes | Request reason |
| 50 | +10 | Yes | Request reason (overtime?) |
| 0 | -40 | Yes | Request reason (absence?) |

---

### Gate 2: Overtime Approval

| Hours | Classification | Supervisor Approval | Finance Approval |
|---|---|---|---|
| 0-40/week | Regular | Yes | No |
| 40-48/week | Overtime | Yes | Yes (budgeted?) |
| 48+/week | Excessive OT | Yes | Yes (director?) |

---

## Process Flow (High-Level)

```
[Employee Enters Hours]
  ↓
[Project/Cost Code Allocation]
  ↓
[Verify: Hours Match Expected]
  ├─ Major variance? → [Request Explanation]
  └─ Normal? → [Send to Supervisor]
  ↓
[Supervisor Review]
  ├─ Hours correct? → YES
  ├─ Check for overtime → [If OT, route to Finance]
  └─ Approve → [LOCKED]
  ↓
[Finance Review (if OT)]
  ├─ Budgeted? → YES
  └─ Approve → [LOCKED]
  ↓
[Payroll Interface]
  ├─ Extract approved hours
  ├─ Calculate pay
  ├─ Post to payroll system
  └─ Record confirmation
  ↓
[Closed]
```

---

## Compliance Checklist for TIME CASE

Before closing:

- [ ] Hours entered by employee
- [ ] Project/cost code allocated correctly
- [ ] Variance from expected identified (if any)
- [ ] Supervisor reviewed and approved
- [ ] Overtime flagged (if applicable)
- [ ] Finance approved overtime (if applicable)
- [ ] Timesheet locked after approval
- [ ] Payroll interface complete
- [ ] Pay calculation verified
- [ ] GL posting recorded
- [ ] Audit log sealed
- [ ] No transition blockers remain

---

## Module Expansion Notes

Detailed implementation will include:
* Leave time accrual and tracking
* Shift differentials
* Bonus and incentive processing
* Multi-state compliance (wage and hour laws)
* Correction procedures (retroactive adjustments)
* Integration with payroll system

---

*VAULT TIME follows the same CASE architecture as PRR. HR-specific rules (overtime, approval) are module-specific.*
