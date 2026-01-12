# VAULT CLERK™ Encoding Specification

**Implementation Contract**  
**Version 1.0 – Final**  
**Authority:** PublicLogic™  
**Runtime Partner:** Polimorphic  
**Date:** January 12, 2026  

**Repo Index:** [INDEX.md](../INDEX.md)

---

## Document Purpose

This is the **authoritative specification** that defines exactly how VAULT CLERK must be implemented.

It is the binding contract between PublicLogic (governance) and Polimorphic (runtime executor).

Polimorphic **shall** implement every rule, state transition, timer, tolling mechanic, validation, enforcement, and logging requirement exactly as written here.  
PublicLogic **shall** validate the delivered system against this document only.

Any deviation requires **prior written approval** from PublicLogic governance authority.

---

## Part I – State Machine

### Permitted States (10 total)

- INTAKE
- COMPLETENESS_CHECK
- ASSESSMENT
- INSPECTION_SCHEDULED
- INSPECTION_PENDING
- INSPECTION_REVIEW
- APPROVAL_REVIEW
- APPROVED
- DENIED
- CLOSED

### Exhaustive Transition Table

Any transition not listed is **prohibited** and **must raise GovernanceViolation**.

| From                   | To                      | Guard Condition (MUST be true)                             | Action / Side Effect                                      | Audit Event                          |
|------------------------|-------------------------|------------------------------------------------------------|-----------------------------------------------------------|--------------------------------------|
| INTAKE                 | COMPLETENESS_CHECK      | Receipt recorded                                           | Validate required documents                               | intake_completed                     |
| COMPLETENESS_CHECK     | ASSESSMENT              | All required documents present & valid                     | Proceed to substantive review                             | completeness_confirmed               |
| COMPLETENESS_CHECK     | INTAKE                  | Documents missing, invalid, or unclear                     | Send clarification request; start tolling                 | clarification_requested              |
| ASSESSMENT             | INSPECTION_SCHEDULED    | inspectionRequired == true                                 | Schedule inspection; set T_INSPECTION due date            | inspection_scheduled                 |
| ASSESSMENT             | APPROVAL_REVIEW         | inspectionRequired == false                                | Skip inspection; proceed to approval                      | inspection_waived                    |
| INSPECTION_SCHEDULED   | INSPECTION_PENDING      | Inspector assigned                                         | Await inspection date                                     | inspection_pending                   |
| INSPECTION_PENDING     | INSPECTION_REVIEW       | Inspection date passed & report uploaded                   | Record results & findings                                 | inspection_completed                 |
| INSPECTION_REVIEW      | APPROVAL_REVIEW         | inspectionResult == "PASS"                                 | Proceed to final approval                                 | inspection_passed                    |
| INSPECTION_REVIEW      | ASSESSMENT              | inspectionResult in ["FAIL", "CONDITIONAL"]                | Request corrections; start tolling                        | inspection_failed                    |
| APPROVAL_REVIEW        | APPROVED                | approvalAuthority grants approval                          | Generate & lock ApprovalLetter                            | approved                             |
| APPROVAL_REVIEW        | DENIED                  | approvalAuthority denies                                   | Generate & lock DenialLetter; cite reason                 | denied                               |
| APPROVED               | CLOSED                  | License certificate issued                                 | Generate & lock LicenseCertificate; set expiry            | license_issued                       |
| DENIED                 | CLOSED                  | Denial becomes final                                       | Close case; open T90 appeal window                        | case_closed_denied                   |

### Non-Negotiable Rules

1. **Deemed Approval** (automatic, no override)
IF current date > T_DECISION due date
AND approvalStatus is null:
Set approvalStatus = "DEEMED_APPROVED"
Generate & lock ApprovalLetter
Generate & lock LicenseCertificate
Set licenseExpiry
Transition to APPROVED
Log: "Deemed approved per statute – T_DECISION exceeded"
text2. **Denial is irreversible** — no transition from DENIED back to any prior state. Reapplication requires new case.

3. **Inspection cannot be skipped** if inspectionRequired == true for the license type.

4. **ApprovalLetter, DenialLetter, LicenseCertificate are locked** on creation — no edits allowed. Corrections require new version (v2) with chain link to original.

---

## Part II – Core Schemas

### CASE (CLERK root object)

```json
{
"$schema": "http://json-schema.org/draft-07/schema#",
"title": "VAULT CLERK CASE",
"type": "object",
"required": [
 "caseId",
 "applicationType",
 "applicantIdentity",
 "intakeRecord",
 "deadlines",
 "currentState",
 "assets",
 "auditLog"
],
"properties": {
 "caseId": {
   "type": "string",
   "pattern": "^clerk-[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}$"
 },
 "applicationType": {
   "enum": ["Contractor", "Plumbing", "Electrical", "Food_Service", "Building_Permit", "Other"]
 },
 "applicantIdentity": {
   "type": "object",
   "required": ["name", "email", "address"],
   "properties": {
     "name": { "type": "string", "minLength": 1 },
     "type": { "enum": ["Individual", "Business"] },
     "email": { "format": "email" },
     "phone": { "type": "string" },
     "address": { "type": "string", "minLength": 5 }
   }
 },
 "intakeRecord": {
   "type": "object",
   "required": ["receivedDate", "effectiveReceiptDate", "channel"],
   "properties": {
     "receivedDate": { "format": "date-time" },
     "effectiveReceiptDate": { "format": "date" },
     "channel": { "enum": ["WEB_FORM", "EMAIL", "MAIL", "IN_PERSON", "FAX"] }
   }
 },
 "deadlines": {
   "type": "object",
   "properties": {
     "t_intake": {
       "properties": {
         "dueDate": { "format": "date" },
         "businessDays": { "const": 2 },
         "status": { "enum": ["OPEN", "MET", "MISSED"] }
       }
     },
     "t_decision": {
       "properties": {
         "dueDate": { "format": "date" },
         "businessDays": { "type": "integer" },
         "status": { "enum": ["OPEN", "MET", "MISSED"] },
         "consequence": { "enum": ["DEEMED_APPROVED", null] }
       }
     },
     "t_inspection": {
       "properties": {
         "dueDate": { "format": "date" },
         "status": { "enum": ["NOT_APPLICABLE", "OPEN", "SCHEDULED", "COMPLETED"] }
       }
     },
     "t90": {
       "properties": {
         "dueDate": { "format": "date" },
         "calendarDays": { "const": 30 }
       }
     }
   }
 },
 "currentState": {
   "enum": ["INTAKE", "COMPLETENESS_CHECK", "ASSESSMENT", "INSPECTION_SCHEDULED", "INSPECTION_PENDING", "INSPECTION_REVIEW", "APPROVAL_REVIEW", "APPROVED", "DENIED", "CLOSED"]
 },
 "approvalAuthority": { "type": "string" },
 "approvalStatus": { "enum": ["PENDING", "APPROVED", "DENIED", "DEEMED_APPROVED", null] },
 "inspectionRequired": { "type": "boolean" },
 "licenseExpiry": { "type": ["string", "null"], "format": "date" },
 "assets": { "type": "array", "items": { "type": "object" } },
 "auditLog": { "type": "array", "items": { "type": "object" } }
}
}
InspectionReport (embedded asset)
JSON{
  "title": "InspectionReport",
  "type": "object",
  "required": ["inspectionDate", "inspector", "result"],
  "properties": {
    "inspectionDate": { "format": "date-time" },
    "inspector": {
      "type": "object",
      "required": ["name", "authority"],
      "properties": {
        "name": { "type": "string" },
        "license": { "type": ["string", "null"] },
        "authority": { "type": "string" }
      }
    },
    "result": { "enum": ["PASS", "FAIL", "CONDITIONAL"] },
    "findings": { "type": "array", "items": { "type": "object" } },
    "correctionDeadline": { "type": ["string", "null"], "format": "date" }
  }
}

Part III – Timer & Enforcement Rules
License-Type Configuration (Normative)






















































Application TypeT_DECISION Business DaysInspection Required?Approval AuthorityDefault License DurationContractor20YesBuilding Inspector2 yearsPlumbing15YesPlumbing Inspector1 yearElectrical15YesElectrical Inspector1 yearFood_Service20YesHealth Department1 yearBuilding_Permit30YesBuilding InspectorProject-specificOther20ConfigurableDesignated authorityConfigurable
T_INTAKE (Completeness Check)

Due: Effective receipt date + 2 business days
If missed → automatic clarification request + tolling starts

T_DECISION (Approval / Denial)

Due: Effective receipt date + business days from table above
Automatic enforcement (no override):textIF current date > T_DECISION due date
   AND approvalStatus is null:
       Set approvalStatus = "DEEMED_APPROVED"
       Generate & lock ApprovalLetter
       Generate & lock LicenseCertificate
       Set licenseExpiry
       Transition to APPROVED
       Log: "Deemed approved per statute – T_DECISION exceeded"

T_INSPECTION (when required)

Due: Configured per license type
If missed → block transition to APPROVAL_REVIEW; notify authority

T90 (Appeal Window)

Starts: Case closure
Duration: 30 calendar days
Late appeal → reject & log

Tolling Rules

Only one active tolling event permitted
Start date inclusive (if business day), end date exclusive
All dates computed in America/New_York timezone

Worked Example:
Clarification sent Jan 20, 2026 (Tue, business day) → response Jan 27, 2026 (Tue)
→ Tolled = 5 business days → all deadlines shift forward by 5 days

Part IV – Deployment & Acceptance Checklist

 License types configured per table (T_DECISION days, inspection required, authority)
 Deemed-approval tested: T_DECISION miss → auto APPROVED + locked letter + license
 Tolling tested: clarification → response → correct 5-day shift verified
 Inspection gating enforced: required → cannot skip; fail → corrections loop
 Locked assets immutable (cannot edit ApprovalLetter/DenialLetter/License)
 Single-active-tolling enforced (second start → error)
 Audit trail complete & append-only
 T90 rejection tested: appeal after 30 days → rejected
