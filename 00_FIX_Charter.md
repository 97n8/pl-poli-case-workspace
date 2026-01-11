# VAULT FIX Module — Charter

**Work Orders, Issue Tracking, Maintenance • Operations**

---

## Module Purpose

VAULT FIX tracks work requests and completions as CASE-based processes.

Examples:
* Facility maintenance requests
* IT help desk tickets
* Work orders for repairs
* Complaint resolution

Each request is one CASE. Process follows from intake through closure.

---

## CASE Model for FIX (Work Order)

### Identity

```json
{
  "CaseID": "fix-2026-0342",
  "CaseType": "FIX",
  "CreatedTimestamp": "2026-02-01T08:00:00Z"
}
```

### Subject

```json
{
  "RequestorIdentity": {
    "Name": "Jane Doe",
    "Department": "Finance"
  },
  "IssueDescription": "Bathroom light not working",
  "Location": "Building A, 2nd Floor, Room 201",
  "Priority": "Normal"
}
```

### Scope

```json
{
  "ScopeDefinition": {
    "WorkType": "Electrical",
    "EstimatedHours": 2,
    "RequiredMaterials": "Fluorescent bulb or ballast"
  }
}
```

### Assets

```json
{
  "Assets": [
    {"AssetType": "WorkOrder", "RetentionClass": "REFERENCE"},
    {"AssetType": "InspectionReport", "RetentionClass": "KEEPER"},
    {"AssetType": "CompletionCertificate", "RetentionClass": "KEEPER", "Locked": true},
    {"AssetType": "LaborLog", "RetentionClass": "REFERENCE"}
  ]
}
```

---

## Module-Specific Rules

### Priority Routing

* Critical: Immediate attention
* High: Within 24 hours
* Normal: Within 5 business days
* Low: Within 2 weeks

CASE records receipt and actual completion, enabling performance measurement.

### Assignment and Tracking

Work can be assigned to contractors or staff. CASE records:
* Who was assigned
* When they started
* When they completed
* What was done

### Completion Verification

Requester (or supervisor) verifies work is complete before CASE closes.

---

## Process Flow (High Level)

```
Intake → Assessment → Assignment → Work Completion → Verification → Closure
```

---

## Decision Tables

### Gate 1: Priority and Response Time

| Priority | Definition | Response Target | Completion Target |
|---|---|---|---|
| Critical | Safety hazard or service down | 2 hours | 24 hours |
| High | Significant impact; workaround available | 4 hours | 5 days |
| Normal | Minor issue; non-urgent | 24 hours | 14 days |
| Low | Cosmetic or improvement | 5 days | 30 days |

---

### Gate 2: Skill Required and Assignment

| Work Type | Skill Required | Assignment Pool |
|---|---|---|
| Electrical | Licensed Electrician | Facilities Electrician or Contractor |
| Plumbing | Licensed Plumber | Facilities Plumber or Contractor |
| HVAC | HVAC Technician | Facilities HVAC or Contractor |
| Carpentry | Carpenter | Facilities Carpenter or Contractor |
| IT | Network/System Admin | IT Staff |

---

### Gate 3: Completion Verification

| Work Type | Verification By | Verification Method |
|---|---|---|
| Electrical | Building Inspector or Licensed Electrician | Inspection + Sign-off |
| Plumbing | Building Inspector or Licensed Plumber | Inspection + Sign-off |
| HVAC | Facilities Manager | Test + Requester confirmation |
| Carpentry | Facilities Manager | Visual + Requester confirmation |
| IT | IT Lead | Testing + Requester confirmation |

---

## Process Flow (High-Level)

```
[Work Request Received]
  ↓
[Priority Assessment]
  ├─ Critical? → [Immediate assignment]
  ├─ High? → [Assign within 4h]
  └─ Normal/Low? → [Assign in order]
  ↓
[Identify Skill Required]
  ↓
[Assign to Technician]
  ├─ In-house available? → [Assign]
  └─ Not available? → [Request contractor quote]
  ↓
[Work Begins]
  ├─ Technician logs time (if tracked)
  ├─ Materials used (if tracked)
  └─ Status updates (if needed)
  ↓
[Work Completed]
  ↓
[Verification]
  ├─ Requester confirms → YES
  └─ Issues found? → [Reopen case / New request]
  ↓
[Completion Certificate] → [LOCKED]
  ↓
[Closed]
```

---

## Compliance Checklist for FIX CASE

Before closing:

- [ ] Request received and described clearly
- [ ] Priority assigned correctly
- [ ] Skill/trade identified
- [ ] Work assigned to technician/contractor
- [ ] Work scheduled and initiated
- [ ] Work completed
- [ ] Verification performed (inspection or test)
- [ ] Requester confirmed completion
- [ ] Completion certificate locked
- [ ] Cost recorded (if applicable)
- [ ] Labor hours recorded (if tracked)
- [ ] Audit log sealed
- [ ] No transition blockers remain

---

## Module Expansion Notes

Detailed implementation will include:
* Contractor management and quotes
* Materials inventory tracking
* PM (preventive maintenance) scheduling
* Cost allocation to departments
* SLA reporting (% critical resolved within target)
* Integration with building management system

---

*VAULT FIX follows the same CASE architecture as PRR. Operations-specific rules (priority routing, verification) are module-specific.*
