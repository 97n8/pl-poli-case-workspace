# VAULT CLERK™ — Encoding Specification

**Implementation Contract • Version 1.0**  
**Authority:** PublicLogic™  
**For Runtime Partners:** Polimorphic

---

## Document Purpose

This specification details **exactly how** to implement VAULT CLERK state machine, data schemas, timers, and validation rules.

Build against this specification. The Governance Architecture is authoritative if you hit ambiguity.

---

## Part I: State Machine Definition

### CLERK State Enum

```json
{
  "states": [
    "INTAKE",
    "COMPLETENESS_CHECK",
    "ASSESSMENT",
    "INSPECTION_SCHEDULED",
    "INSPECTION_PENDING",
    "INSPECTION_REVIEW",
    "APPROVAL_REVIEW",
    "APPROVED",
    "DENIED",
    "CLOSED"
  ]
}
```

### State Transition Matrix

| From | To | Condition | Action | Audit |
|---|---|---|---|---|
| INTAKE | COMPLETENESS_CHECK | Receipt recorded | Validate all required documents received | LogEntry: intake_completed |
| COMPLETENESS_CHECK | ASSESSMENT | All docs present | Proceed to assessment | LogEntry: completeness_confirmed |
| COMPLETENESS_CHECK | INTAKE | Clarification needed | Request missing documents; toll clock | LogEntry: clarification_requested |
| ASSESSMENT | INSPECTION_SCHEDULED | InspectionRequired = true | Schedule inspection; set T_INSPECTION | LogEntry: inspection_scheduled |
| ASSESSMENT | APPROVAL_REVIEW | InspectionRequired = false | Skip inspection; proceed to approval | LogEntry: inspection_waived |
| INSPECTION_SCHEDULED | INSPECTION_PENDING | Inspector assigned | Wait for inspection date | LogEntry: inspection_pending |
| INSPECTION_PENDING | INSPECTION_REVIEW | Inspection completed | Record inspection results | LogEntry: inspection_completed |
| INSPECTION_REVIEW | APPROVAL_REVIEW | Inspection PASS | Proceed to approval | LogEntry: inspection_passed |
| INSPECTION_REVIEW | ASSESSMENT | Inspection FAIL/CONDITIONAL | Request corrections from applicant; toll | LogEntry: inspection_failed |
| APPROVAL_REVIEW | APPROVED | ApprovalAuthority approves | Generate ApprovalLetter (LOCKED) | LogEntry: approved |
| APPROVAL_REVIEW | DENIED | ApprovalAuthority denies | Generate DenialLetter (LOCKED); cite reason | LogEntry: denied |
| APPROVED | CLOSED | License issued | License certificate generated (LOCKED); set LicenseExpiry | LogEntry: license_issued |
| DENIED | CLOSED | Denial final | Case closed; appeal window opens | LogEntry: case_closed_denied |

### Non-Negotiable Rules

1. **Deemed Approval Logic:**
   ```
   IF CurrentDate > T_DECISION_DueDate
   AND ApprovalStatus == null (not set):
     ApprovalStatus = APPROVED
     ApprovalLetter generated automatically
     License issued
     LogEvent: "Deemed approved per statute"
   ```

2. **Denial is Irreversible:**
   - Once DENIED, cannot transition back to APPROVAL_REVIEW
   - Reapplication required (new CASE)

3. **Inspection is Mandatory (if required):**
   - Cannot skip INSPECTION_SCHEDULED → INSPECTION_PENDING → INSPECTION_REVIEW
   - Only way to skip is explicit InspectionRequired = false (configured per license type)

4. **Approval/Denial Letters are LOCKED:**
   - Cannot edit reason once letter generated
   - Corrections require new letter (version 2)

---

## Part II: JSON Schemas

### CASE Schema (CLERK)

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "CLERK CASE",
  "type": "object",
  "required": [
    "caseId",
    "caseType",
    "applicationType",
    "applicantIdentity",
    "intakeRecord",
    "deadlines",
    "assets",
    "auditLog"
  ],
  "properties": {
    "caseId": {
      "type": "string",
      "description": "Immutable UUID",
      "pattern": "^clerk-[a-f0-9-]{36}$"
    },
    "caseType": {
      "type": "string",
      "enum": ["CLERK"],
      "description": "Always CLERK for this module"
    },
    "applicationType": {
      "type": "string",
      "enum": ["Contractor", "Plumbing", "Electrical", "Food", "Building", "Other"],
      "description": "License type"
    },
    "applicantIdentity": {
      "type": "object",
      "required": ["name", "email", "address"],
      "properties": {
        "name": {
          "type": "string",
          "description": "Applicant or business name"
        },
        "type": {
          "type": "string",
          "enum": ["Individual", "Business"],
          "description": "Type of applicant"
        },
        "email": {
          "type": "string",
          "format": "email"
        },
        "phone": {
          "type": "string"
        },
        "address": {
          "type": "string"
        }
      }
    },
    "intakeRecord": {
      "type": "object",
      "required": ["receivedDate", "effectiveReceiptDate", "channel"],
      "properties": {
        "receivedDate": {
          "type": "string",
          "format": "date-time",
          "description": "When application was actually received"
        },
        "effectiveReceiptDate": {
          "type": "string",
          "format": "date",
          "description": "Computed; shifts if weekend/holiday"
        },
        "channel": {
          "type": "string",
          "enum": ["WEB_FORM", "EMAIL", "MAIL", "IN_PERSON", "FAX"],
          "description": "How application was submitted"
        }
      }
    },
    "deadlines": {
      "type": "object",
      "properties": {
        "t_intake": {
          "type": "object",
          "properties": {
            "dueDate": { "type": "string", "format": "date" },
            "businessDays": { "type": "integer", "default": 2 },
            "status": { "type": "string", "enum": ["OPEN", "MET", "MISSED"] }
          }
        },
        "t_decision": {
          "type": "object",
          "properties": {
            "dueDate": { "type": "string", "format": "date" },
            "businessDays": { "type": "integer" },
            "status": { "type": "string", "enum": ["OPEN", "MET", "MISSED"] },
            "consequence": {
              "type": "string",
              "enum": ["DEEMED_APPROVED", "APPLICATION_DENIED"],
              "description": "What happens if deadline missed"
            }
          }
        },
        "t_inspection": {
          "type": "object",
          "properties": {
            "dueDate": { "type": "string", "format": "date" },
            "businessDays": { "type": "integer" },
            "status": { "type": "string", "enum": ["NOT_APPLICABLE", "OPEN", "SCHEDULED", "COMPLETED"] }
          }
        },
        "t90": {
          "type": "object",
          "properties": {
            "dueDate": { "type": "string", "format": "date" },
            "calendarDays": { "type": "integer", "default": 30 },
            "activatesAt": { "type": "string", "enum": ["CASE_CLOSED"] }
          }
        }
      }
    },
    "currentState": {
      "type": "string",
      "enum": ["INTAKE", "COMPLETENESS_CHECK", "ASSESSMENT", "INSPECTION_SCHEDULED", "INSPECTION_PENDING", "INSPECTION_REVIEW", "APPROVAL_REVIEW", "APPROVED", "DENIED", "CLOSED"]
    },
    "approvalAuthority": {
      "type": "string",
      "description": "Who approves (Building Inspector, Health Director, etc.)"
    },
    "approvalStatus": {
      "type": "string",
      "enum": ["PENDING", "APPROVED", "DENIED", "DEEMED_APPROVED", null],
      "description": "Approval decision (null until made)"
    },
    "inspectionRequired": {
      "type": "boolean",
      "description": "Does this license type require inspection?"
    },
    "licenseExpiry": {
      "type": "string",
      "format": "date",
      "description": "License expiration date (null if denied)"
    },
    "assets": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "assetId": { "type": "string" },
          "assetType": {
            "type": "string",
            "enum": ["Application", "SupportingDocuments", "ApprovalLetter", "DenialLetter", "InspectionReport", "LicenseCertificate", "RenewalNotice"]
          },
          "retentionClass": {
            "type": "string",
            "enum": ["KEEPER", "REFERENCE", "TRANSACTIONAL"]
          },
          "isLocked": { "type": "boolean" },
          "lockedTimestamp": { "type": "string", "format": "date-time" },
          "versionChain": { "type": "array" }
        }
      }
    },
    "auditLog": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "eventId": { "type": "string" },
          "timestamp": { "type": "string", "format": "date-time" },
          "actor": { "type": "string" },
          "action": { "type": "string" },
          "stateFrom": { "type": "string" },
          "stateTo": { "type": "string" },
          "reason": { "type": "string" }
        }
      }
    }
  }
}
```

### InspectionReport Schema

```json
{
  "title": "InspectionReport",
  "type": "object",
  "required": ["inspectionDate", "inspector", "result"],
  "properties": {
    "inspectionDate": {
      "type": "string",
      "format": "date-time"
    },
    "inspector": {
      "type": "object",
      "properties": {
        "name": { "type": "string" },
        "license": { "type": "string" },
        "authority": { "type": "string" }
      }
    },
    "result": {
      "type": "string",
      "enum": ["PASS", "FAIL", "CONDITIONAL"],
      "description": "Conditional = pass with corrections required"
    },
    "findings": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "category": { "type": "string" },
          "severity": { "type": "string", "enum": ["CRITICAL", "MAJOR", "MINOR"] },
          "description": { "type": "string" },
          "requiresCorrection": { "type": "boolean" }
        }
      }
    },
    "correctionDeadline": {
      "type": "string",
      "format": "date",
      "description": "If CONDITIONAL, when corrections due"
    }
  }
}
```

---

## Part III: Timer Algorithms

### Timer: T_INTAKE

**Definition:** Application completeness verification deadline

```
EffectiveReceiptDate = ComputeEffectiveReceiptDate(received_date)
T_INTAKE_DueDate = EffectiveReceiptDate + 2 business days

ENFORCE:
  IF CurrentDate > T_INTAKE_DueDate
     AND ApplicationStatus == "PENDING":
    Consequence: REQUEST_CLARIFICATION
    Action: Email applicant; clock tolled
    LogEvent: "T_INTAKE exceeded; clarification requested"
```

### Timer: T_DECISION

**Definition:** Approval/denial decision deadline

```
T_DECISION_DueDate = EffectiveReceiptDate + X business days
  where X = Application Type Specific (Contractor: 20, Plumbing: 15, Food: 20, etc.)

ENFORCE:
  IF CurrentDate > T_DECISION_DueDate
     AND ApprovalStatus == null (not set):
    Consequence: DEEMED_APPROVED
    Action:
      1. Set ApprovalStatus = APPROVED
      2. Generate ApprovalLetter automatically
      3. Set LicenseExpiry = date + license duration (e.g., 3 years)
      4. Transition to APPROVED
      5. Log: "T_DECISION deadline exceeded; deemed approved per statute"
```

### Timer: T_INSPECTION

**Definition:** Inspection completion deadline (if inspection required)

```
T_INSPECTION_DueDate = EffectiveReceiptDate + X business days
  where X = Application Type Specific (varies)

ENFORCE:
  IF CurrentDate > T_INSPECTION_DueDate
     AND InspectionRequired == true
     AND InspectionStatus != "COMPLETED":
    Consequence: GATE_BLOCKED
    Action:
      1. Block transition to APPROVAL_REVIEW
      2. Add TransitionBlocker: "Inspection incomplete"
      3. Notify ApprovalAuthority
      4. Log: "T_INSPECTION deadline exceeded; inspection required before approval"
```

### Timer: T90 (Appeal Window)

**Definition:** Appellant right to appeal denied license

```
Activated: At CASE closure (DENIED or APPROVED → CLOSED)
T90_DueDate = Case Closure Date + 30 calendar days

ENFORCE:
  IF AppealAttempted == true
     AND CurrentDate > T90_DueDate:
    Action:
      1. Reject appeal request
      2. Message: "Appeal window has closed; T90 exceeded"
      3. Log: "Appeal rejected; T90 expired"
```

---

## Part IV: Tolling Protocol

### Tolling Scenario 1: Clarification Requested

```
Event: Applicant lacks required documents
Action:
  1. Email sent requesting document (EmailStep)
  2. Tolling starts
  3. T_INTAKE clock paused
  4. TollingEvent logged: 
     {
       "reason": "CLARIFICATION_REQUESTED",
       "startDate": "2026-01-20",
       "asset": "CLARIFICATION_EMAIL"
     }

Resume Trigger: Applicant submits documents
Action:
  1. Documents received
  2. Tolling ends
  3. Tolled days calculated: "2026-01-20" to "2026-01-25" = 5 business days
  4. T_INTAKE_DueDate recalculated: original due date + 5 days
  5. TollingEvent updated:
     {
       "endDate": "2026-01-25",
       "tolledBusinessDays": 5,
       "resumeAsset": "CLARIFICATION_RESPONSE"
     }
```

### Tolling Scenario 2: Inspection Correction

```
Event: Inspection fails; corrections required
Action:
  1. Correction deadline set (T_CORRECTION_DueDate)
  2. T_DECISION clock tolled
  3. TollingEvent logged:
     {
       "reason": "INSPECTION_CORRECTIONS",
       "startDate": "2026-01-22",
       "asset": "INSPECTION_REPORT",
       "correctionDeadline": "2026-02-05"
     }

Resume Trigger: Applicant corrects; re-inspection passes
Action:
  1. Re-inspection completed; results documented
  2. Tolling ends
  3. Tolled days calculated: "2026-01-22" to "2026-02-10" = 15 business days
  4. T_DECISION_DueDate recalculated: original due + 15 days
  5. TollingEvent updated:
     {
       "endDate": "2026-02-10",
       "tolledBusinessDays": 15,
       "resumeAsset": "RE_INSPECTION_REPORT"
     }
```

---

## Part V: Validation Rules

### Rule: ApplicationType Must Be Configured

```
Validation:
  IF ApplicationType not in configured license types:
    REJECT
    Error: "License type not configured in this municipality"
    HttpStatus: 400

Example Configured Types:
  - Contractor
  - Plumbing
  - Electrical
  - Food Service
```

### Rule: ApprovalAuthority Must Be Set

```
Validation:
  IF ApplicationType configured
     AND ApprovalAuthority == null:
    REJECT
    Error: "Approval authority not designated for this license type"
    HttpStatus: 400
```

### Rule: InspectionRequired Determines Flow

```
Validation:
  IF ApplicationType == "Contractor":
    InspectionRequired = true (mandatory)
    InspectionAuthority = "Building Inspector"
  ELSE IF ApplicationType == "Food":
    InspectionRequired = true
    InspectionAuthority = "Health Department"
  ELSE IF ApplicationType == "Building Permit":
    InspectionRequired = true
    InspectionAuthority = "Building Inspector"
  (etc.)
```

### Rule: Deemed Approval Deadline Logic

```
Validation (T_DECISION):
  IF ApplicationType requires T_DECISION
     AND CurrentDate > T_DECISION_DueDate
     AND ApprovalStatus == null:
    ENFORCE: Deemed Approval
    Action:
      1. Generate ApprovalLetter automatically
      2. Set ApprovalStatus = APPROVED
      3. Transition to APPROVED state
      4. Log: automatic approval due to deadline miss
```

---

## Part VI: Error Handling

### Error Code: E_INSPECTION_OVERDUE

```
Condition: T_INSPECTION deadline exceeded; inspection not completed

Handling:
  Category: HIGH (blocks approval)
  HttpStatus: 409 (Conflict)
  Response:
    {
      "error_code": "E_INSPECTION_OVERDUE",
      "message": "Inspection completion deadline exceeded",
      "case_id": "clerk-2026-0145",
      "deadline": "2026-02-15",
      "days_overdue": 5,
      "action": "Request extension from applicant or complete inspection"
    }
  
  Recovery:
    1. Applicant requests extension
    2. ApprovalAuthority grants extension (optional)
    3. New T_INSPECTION_DueDate set
    4. TollingEvent logged
    5. Transition proceeds
```

### Error Code: E_MISSING_INSPECTION_REPORT

```
Condition: Attempt to transition to APPROVAL_REVIEW without inspection report

Handling:
  Category: CRITICAL (blocks transition)
  HttpStatus: 409
  Response:
    {
      "error_code": "E_MISSING_INSPECTION_REPORT",
      "message": "Inspection report required before approval",
      "case_id": "clerk-2026-0145",
      "action": "Complete inspection and upload report"
    }
```

### Error Code: E_CANNOT_EDIT_LOCKED_DENIAL

```
Condition: Attempt to edit DenialLetter after locked

Handling:
  Category: CRITICAL (governance violation)
  HttpStatus: 403 (Forbidden)
  Response:
    {
      "error_code": "E_CANNOT_EDIT_LOCKED_DENIAL",
      "message": "Denial letter is locked and cannot be edited",
      "case_id": "clerk-2026-0145",
      "asset_id": "denial-letter-001",
      "locked_timestamp": "2026-02-15T14:32:00Z",
      "action": "If correction needed, create version 2 of denial letter"
    }
  
  Recovery:
    1. Create new DenialLetter version 2
    2. Set SupersedesVersion: 1
    3. Lock new version
    4. Audit trail shows both versions
```

---

## Part VII: Deployment Checklist

Before go-live:

- [ ] All license types configured (ApplicationType, InspectionRequired, DecisionDeadline, ApprovalAuthority)
- [ ] Timer computation tested for each license type (T_INTAKE, T_DECISION, T_INSPECTION, T90)
- [ ] Deemed approval logic tested (deadline miss triggers auto-approval)
- [ ] Tolling tested (clarification request, inspection corrections)
- [ ] State machine tested (all valid transitions, all gates enforced)
- [ ] Inspection report schema validated
- [ ] Approval/Denial letter templates created and locked
- [ ] Appeal window (T90) tested
- [ ] Audit logging verified (completeness, immutability)
- [ ] Error handling tested (all error codes)

---

*This Encoding Specification is the build contract. Implement exactly as specified. Ambiguities escalate to PublicLogic.*
