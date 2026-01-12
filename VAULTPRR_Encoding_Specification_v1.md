# VAULTPRR™ Encoding Specification

**Authoritative Runtime Contract**  
**Version:** 1.0 – Final  
**Authority:** PublicLogic™  
**Governing Law:** M.G.L. c. 66 § 10 • 950 CMR 32.00  
**Date:** January 12, 2026  

**Repo Index:** [INDEX.md](INDEX.md)  

---

## Document Purpose

This is the **authoritative contract** between PublicLogic (governance authority) and Polimorphic (runtime executor).  

It defines the required implementation:  
- Executable state machine with 22 states and transitions  
- Data schemas with validation rules  
- Deadline computation algorithms  
- Tolling protocol for pausing and resuming the clock  
- Error and timeout handling  
- Audit log schema  

**Compliance Requirement:**  
Polimorphic **shall** build exactly to this specification.  
PublicLogic **shall** validate the runtime against this document.  
Deviations require **prior written approval** from PublicLogic.  

---

## 1. State Machine

### 1.1 Enumerated States

| StateID | State Name              | Category  | Description / Responsibility |
|---------|-------------------------|-----------|------------------------------|
| S000   | CREATED                 | System    | Case instantiated            |
| S100   | INTAKE                  | Form      | Requester submits PRR        |
| S200   | TIMER_COMPUTE           | System    | Deadlines calculated         |
| S300   | ASSESSMENT              | Review    | RAO evaluates scope          |
| S310   | CLARIFICATION_SENT      | Email     | Clarification dispatched     |
| S320   | CLARIFICATION_WAIT      | Wait      | Clock paused for response    |
| S330   | FORECAST_REVIEW         | Decision  | T25 forecast evaluation      |
| S340   | EXTENSION_AGREEMENT     | Document  | Requester agrees to extension|
| S350   | PETITION_FILED          | Document  | Petition to Supervisor       |
| S400   | GATHER                  | Manual    | Records search/collection    |
| S500   | EXEMPTION_REVIEW        | Review    | Exemption analysis           |
| S510   | REDACTION               | Manual    | Redactions applied           |
| S520   | COUNSEL_REVIEW          | Gate      | Optional legal review        |
| S600   | PACKAGE                 | Document  | Response package assembled   |
| S700   | FEE_ASSESSMENT          | Decision  | Fee check (T10 enforced)     |
| S710   | INVOICE                 | Document  | Fee estimate/invoice         |
| S720   | PAYMENT_WAIT            | Wait      | Awaiting payment             |
| S800   | DELIVERY                | Manual    | Records transmitted          |
| S900   | CLOSED                  | Terminal  | Case complete                |
| S910   | CLOSED_NO_RECORDS       | Terminal  | No responsive records        |
| S920   | CLOSED_WITHHELD         | Terminal  | Full withholding             |
| S930   | CLOSED_NON_PAYMENT      | Terminal  | No payment received          |
| S990   | ERROR                   | Terminal  | System failure               |
| S991   | TIMEOUT                 | Terminal  | Exceeded wait threshold      |

### 1.2 Valid Transitions

The following transitions are exhaustive. Any other is invalid and **must** be rejected.

```
S000 → S100
S100 → S200
S200 → S300
S300 → S310 | S330 | S400 | S910
S310 → S320
S320 → S300 | S991
S330 → S340 | S350 | S400
S340 → S400
S350 → S400
S400 → S500
S500 → S510 | S600 | S920
S510 → S520 | S600
S520 → S510 | S600
S600 → S700
S700 → S710 | S800
S710 → S720
S720 → S800 | S930 | S991
S800 → S900

ANY → S990 (on system error)
```

### 1.3 Transition Guards

| Transition | Guard Condition | Enforcement |
|------------|-----------------|-------------|
| S300 → S310 | scope.clarificationNeeded == true | Block if scope is clear |
| S300 → S910 | assessment.responsiveRecords == false | Requires RAO attestation |
| S320 → S300 | clarification.responseReceived == true | Must have response payload |
| S320 → S991 | elapsed(S320.entryTime) > CLARIFICATION_TIMEOUT | Default: 30 calendar days |
| S330 → S340 | forecast.completionDate > T25 AND requester.agreesToExtension == true | Requires signed agreement |
| S330 → S350 | forecast.completionDate > T25 AND requester.agreesToExtension == false | Requires petition |
| S500 → S920 | exemptionReview.allExempt == true | Requires withholding letter |
| S520 → S510 | counselReview.revisionsRequired == true | Revision notes required |
| S700 → S710 | T10.met == true AND fees.required == true | Fee estimate > $0 |
| S700 → S800 | T10.met == false OR fees.required == false | Fees waived or not applicable |
| S720 → S930 | elapsed(S720.entryTime) > PAYMENT_TIMEOUT | Default: 30 calendar days |

---

## 2. Data Schemas

All schemas are normative. Fields marked "required" are mandatory.

### 2.1 CASE

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "required": ["caseId", "requester", "intake", "deadlines", "status"],
  "properties": {
    "caseId": {
      "type": "string",
      "format": "uuid",
      "description": "Immutable unique identifier"
    },
    "requester": { "$ref": "#/definitions/RequesterIdentity" },
    "intake": { "$ref": "#/definitions/IntakeRecord" },
    "scope": { "$ref": "#/definitions/ScopeDefinition" },
    "deadlines": { "$ref": "#/definitions/DeadlineModel" },
    "feesAllowed": {
      "type": "boolean",
      "default": true,
      "description": "Set to false if T10 missed"
    },
    "status": { "$ref": "#/definitions/CaseStatus" },
    "closureReason": {
      "type": "string",
      "enum": ["DELIVERED", "NO_RECORDS", "WITHHELD", "NON_PAYMENT", "WITHDRAWN", "ERROR", "TIMEOUT"]
    },
    "assets": {
      "type": "array",
      "items": { "$ref": "#/definitions/Asset" }
    },
    "auditLog": {
      "type": "array",
      "items": { "$ref": "#/definitions/LogEntry" }
    }
  }
}
```

### 2.2 RequesterIdentity

```json
{
  "definitions": {
    "RequesterIdentity": {
      "type": "object",
      "required": ["name", "contactMethod"],
      "properties": {
        "name": {
          "type": "string",
          "minLength": 1,
          "maxLength": 255
        },
        "organization": {
          "type": "string",
          "maxLength": 255
        },
        "email": {
          "type": "string",
          "format": "email"
        },
        "phone": {
          "type": "string",
          "pattern": "^[0-9\\-\\+\\(\\)\\s]+$"
        },
        "mailingAddress": {
          "type": "string",
          "maxLength": 500
        },
        "contactMethod": {
          "type": "string",
          "enum": ["EMAIL", "MAIL", "FAX", "IN_PERSON"]
        },
        "isAnonymous": {
          "type": "boolean",
          "default": false
        }
      }
    }
  }
}
```

### 2.3 IntakeRecord

```json
{
  "definitions": {
    "IntakeRecord": {
      "type": "object",
      "required": ["receivedTimestamp", "channel", "rawRequestText"],
      "properties": {
        "receivedTimestamp": {
          "type": "string",
          "format": "date-time"
        },
        "effectiveReceiptDate": {
          "type": "string",
          "format": "date"
        },
        "channel": {
          "type": "string",
          "enum": ["WEB_FORM", "EMAIL", "MAIL", "FAX", "IN_PERSON", "PHONE"]
        },
        "rawRequestText": {
          "type": "string",
          "minLength": 1
        },
        "attachments": {
          "type": "array",
          "items": { "type": "string", "format": "uuid" }
        }
      }
    }
  }
}
```

### 2.4 ScopeDefinition

```json
{
  "definitions": {
    "ScopeDefinition": {
      "type": "object",
      "required": ["version", "description"],
      "properties": {
        "version": {
          "type": "integer",
          "minimum": 1
        },
        "description": {
          "type": "string",
          "minLength": 1
        },
        "dateRangeStart": { "type": "string", "format": "date" },
        "dateRangeEnd": { "type": "string", "format": "date" },
        "departments": { "type": "array", "items": { "type": "string" } },
        "keywords": { "type": "array", "items": { "type": "string" } },
        "clarificationHistory": {
          "type": "array",
          "items": {
            "type": "object",
            "properties": {
              "version": { "type": "integer" },
              "timestamp": { "type": "string", "format": "date-time" },
              "description": { "type": "string" },
              "clarificationRequest": { "type": "string" },
              "requesterResponse": { "type": "string" }
            }
          }
        }
      }
    }
  }
}
```

### 2.5 DeadlineModel

```json
{
  "definitions": {
    "DeadlineModel": {
      "type": "object",
      "required": ["effectiveReceiptDate", "T10", "T20", "T25", "T90"],
      "properties": {
        "effectiveReceiptDate": { "type": "string", "format": "date" },
        "T10": {
          "type": "object",
          "properties": {
            "dueDate": { "type": "string", "format": "date" },
            "met": { "type": "boolean" },
            "metTimestamp": { "type": "string", "format": "date-time" }
          }
        },
        "T20": {
          "type": "object",
          "properties": {
            "windowCloses": { "type": "string", "format": "date" },
            "petitionFiled": { "type": "boolean" }
          }
        },
        "T25": {
          "type": "object",
          "properties": {
            "dueDate": { "type": "string", "format": "date" },
            "extendedDueDate": { "type": "string", "format": "date" },
            "extensionGrantedBy": {
              "type": "string",
              "enum": ["REQUESTER_AGREEMENT", "SUPERVISOR_GRANT"]
            }
          }
        },
        "T90": {
          "type": "object",
          "properties": {
            "windowCloses": { "type": "string", "format": "date" },
            "appealFiled": { "type": "boolean" }
          }
        },
        "tollingEvents": {
          "type": "array",
          "items": { "$ref": "#/definitions/TollingEvent" }
        },
        "businessDaysTolled": {
          "type": "integer",
          "minimum": 0
        }
      }
    }
  }
}
```

### 2.6 TollingEvent

```json
{
  "definitions": {
    "TollingEvent": {
      "type": "object",
      "required": ["eventId", "reason", "startTimestamp", "status"],
      "properties": {
        "eventId": { "type": "string", "format": "uuid" },
        "reason": {
          "type": "string",
          "enum": ["CLARIFICATION", "REQUESTER_DELAY", "SUPERVISOR_EXTENSION"]
        },
        "startTimestamp": { "type": "string", "format": "date-time" },
        "endTimestamp": { "type": "string", "format": "date-time" },
        "status": {
          "type": "string",
          "enum": ["ACTIVE", "CLOSED"]
        },
        "businessDaysTolled": { "type": "integer", "minimum": 0 },
        "triggerAssetId": { "type": "string", "format": "uuid" },
        "resumeAssetId": { "type": "string", "format": "uuid" }
      }
    }
  }
}
```

### 2.7 Asset

```json
{
  "definitions": {
    "Asset": {
      "type": "object",
      "required": ["assetId", "assetType", "retentionClass", "createdTimestamp"],
      "properties": {
        "assetId": { "type": "string", "format": "uuid" },
        "assetType": {
          "type": "string",
          "enum": [
            "ORIGINAL_REQUEST",
            "CLARIFICATION_REQUEST",
            "CLARIFICATION_RESPONSE",
            "SCOPE_DOCUMENT",
            "SEARCH_LOG",
            "RESPONSIVE_RECORD",
            "REDACTED_RECORD",
            "EXEMPTION_LOG",
            "NO_RECORDS_LETTER",
            "WITHHOLDING_LETTER",
            "RESPONSE_PACKAGE",
            "EXTENSION_AGREEMENT",
            "PETITION",
            "FEE_ESTIMATE",
            "INVOICE",
            "PAYMENT_RECEIPT",
            "DELIVERY_CONFIRMATION"
          ]
        },
        "retentionClass": {
          "type": "string",
          "enum": ["KEEPER", "REFERENCE", "TRANSACTIONAL"]
        },
        "locked": { "type": "boolean", "default": false },
        "createdTimestamp": { "type": "string", "format": "date-time" },
        "createdBy": { "type": "string" },
        "versionChain": {
          "type": "array",
          "items": { "type": "string", "format": "uuid" }
        },
        "fileReference": { "type": "string" },
        "mimeType": { "type": "string" },
        "hash": { "type": "string" },
        "exemptions": {
          "type": "array",
          "items": { "$ref": "#/definitions/ExemptionTag" }
        },
        "redactions": {
          "type": "array",
          "items": { "$ref": "#/definitions/RedactionTag" }
        }
      }
    }
  }
}
```

### 2.8 ExemptionTag

```json
{
  "definitions": {
    "ExemptionTag": {
      "type": "object",
      "required": ["exemptionCode", "citation"],
      "properties": {
        "exemptionCode": { "type": "string", "pattern": "^[A-Z]{2,4}-[0-9]{1,3}[A-Za-z]?$" },
        "citation": { "type": "string" },
        "description": { "type": "string" },
        "appliedBy": { "type": "string" },
        "appliedTimestamp": { "type": "string", "format": "date-time" },
        "counselApproved": { "type": "boolean" }
      }
    }
  }
}
```

### 2.9 RedactionTag

```json
{
  "definitions": {
    "RedactionTag": {
      "type": "object",
      "required": ["redactionId", "exemptionCode", "location"],
      "properties": {
        "redactionId": { "type": "string", "format": "uuid" },
        "exemptionCode": { "type": "string" },
        "location": {
          "type": "object",
          "properties": {
            "pageNumber": { "type": "integer" },
            "boundingBox": {
              "type": "object",
              "properties": {
                "x": { "type": "number" },
                "y": { "type": "number" },
                "width": { "type": "number" },
                "height": { "type": "number" }
              }
            }
          }
        },
        "appliedBy": { "type": "string" },
        "appliedTimestamp": { "type": "string", "format": "date-time" }
      }
    }
  }
}
```

### 2.10 CaseStatus

```json
{
  "definitions": {
    "CaseStatus": {
      "type": "object",
      "required": ["currentState", "lastTransition"],
      "properties": {
        "currentState": { "type": "string", "pattern": "^S[0-9]{3}$" },
        "lastTransition": { "type": "string", "format": "date-time" },
        "assignedTo": { "type": "string" },
        "flags": {
          "type": "array",
          "items": {
            "type": "string",
            "enum": ["URGENT", "MEDIA", "LITIGATION_HOLD", "COMPLEX", "ESCALATED"]
          }
        }
      }
    }
  }
}
```

### 2.11 LogEntry

```json
{
  "definitions": {
    "LogEntry": {
      "type": "object",
      "required": ["logId", "timestamp", "actor", "action", "stateFrom", "stateTo"],
      "properties": {
        "logId": { "type": "string", "format": "uuid" },
        "timestamp": { "type": "string", "format": "date-time" },
        "actor": {
          "type": "object",
          "properties": {
            "actorId": { "type": "string" },
            "actorType": {
              "type": "string",
              "enum": ["SYSTEM", "RAO", "DELEGATE", "REQUESTER", "COUNSEL", "SUPERVISOR"]
            },
            "actorName": { "type": "string" }
          }
        },
        "action": {
          "type": "string",
          "enum": [
            "CASE_CREATED",
            "STATE_TRANSITION",
            "ASSET_CREATED",
            "ASSET_LOCKED",
            "TIMER_COMPUTED",
            "TIMER_TOLLED",
            "TIMER_RESUMED",
            "FEE_WAIVED",
            "ASSIGNMENT_CHANGED",
            "FLAG_ADDED",
            "FLAG_REMOVED",
            "CASE_CLOSED",
            "ERROR_LOGGED"
          ]
        },
        "stateFrom": { "type": "string", "pattern": "^S[0-9]{3}$" },
        "stateTo": { "type": "string", "pattern": "^S[0-9]{3}$" },
        "assetsAffected": {
          "type": "array",
          "items": { "type": "string", "format": "uuid" }
        },
        "metadata": {
          "type": "object",
          "additionalProperties": true
        }
      }
    }
  }
}
```

---

## 3. Timer Computation

### 3.1 Business Day Calendar

- Load at runtime from authoritative source  
- Massachusetts legal holidays (e.g., New Year's Day, MLK Day, Presidents' Day, etc.)  
- Municipality-specific closures (e.g., town meetings, emergencies)  

### 3.2 Effective Receipt Date

If received on non-business day → next business day.

### 3.3 Deadline Computation

From effective receipt:  
- T10: 10 business days (initial response)  
- T20: 20 business days (petition window)  
- T25: 25 business days (max production)  

### 3.4 Tolling Protocol

Start: on clarification sent, requester delay, supervisor extension  
End: on complete response  
Count: business days tolled (inclusive start if business day, exclusive end)  
Recompute: add tolled days to all deadlines  

---

## 4. Validation Rules

### 4.1 Step Entry Conditions

| Step | Entry Conditions |
|------|------------------|
| S100 INTAKE | Case exists in S000 |
| S200 TIMER_COMPUTE | Intake complete (requester, channel, raw text) |
| S300 ASSESSMENT | Deadlines computed; assignedTo set |
| S310 CLARIFICATION_SENT | Assessment started; clarification needed |
| S400 GATHER | Assessment complete; scope defined; forecast ≤ T25 or extension/petition filed |
| S500 EXEMPTION_REVIEW | Gather complete; responsive records or search log |
| S600 PACKAGE | Exemption review complete; redactions applied |
| S700 FEE_ASSESSMENT | Package created & locked |
| S800 DELIVERY | Fee assessment complete; payment or waiver |
| S900 CLOSED | Delivery confirmed |

### 4.2 Step Exit Conditions

| Step | Exit Conditions |
|------|-----------------|
| S100 INTAKE | Required fields populated; original request asset |
| S200 TIMER_COMPUTE | T10/T20/T25 due dates stored |
| S300 ASSESSMENT | Decision: proceed, clarify, or no records |
| S320 CLARIFICATION_WAIT | Response received or timeout |
| S400 GATHER | Search log; records identified or none |
| S500 EXEMPTION_REVIEW | Exemptions determined for all records |
| S600 PACKAGE | Package asset created & locked |
| S710 INVOICE | Invoice asset created; total calculated |
| S720 PAYMENT_WAIT | Payment received or timeout |
| S800 DELIVERY | Delivery confirmation asset created |

### 4.3 Asset Validation

| Asset Type | Required Fields | Lock Behavior |
|------------|-----------------|---------------|
| ORIGINAL_REQUEST | fileReference, hash | Locked on creation |
| NO_RECORDS_LETTER | fileReference, hash | Locked on creation |
| WITHHOLDING_LETTER | fileReference, hash, exemptions[] | Locked on creation |
| RESPONSE_PACKAGE | fileReference, hash, exemptions[], redactions[] | Locked on creation |
| EXTENSION_AGREEMENT | fileReference, hash | Locked on creation |
| PETITION | fileReference, hash | Locked on creation |
| INVOICE | total, lineItems[] | Mutable until paid |
| PAYMENT_RECEIPT | amount, paymentMethod, transactionId | Locked on creation |

---

## 5. Error & Timeout Handling

### 5.1 Error States

| Code | Trigger | Recovery |
|------|---------|----------|
| ERR_001 | Database write failure | Retry 3x, then S990 + manual flag |
| ERR_002 | Asset storage failure | Retry 3x, then S990 + manual flag |
| ERR_003 | Calendar unavailable | Use cache; flag for review |
| ERR_004 | Invalid transition | Reject; log; alert |
| ERR_005 | Missing field on exit | Block; return errors |
| ERR_006 | Hash mismatch | Block; investigate integrity |

### 5.2 Timeout Thresholds

| Wait State | Timeout | Outcome |
|------------|---------|---------|
| S320 CLARIFICATION_WAIT | 30 calendar days | S991 + log + notify RAO |
| S720 PAYMENT_WAIT | 30 calendar days | S930 CLOSED_NON_PAYMENT |

### 5.3 Timeout Protocol

Run daily scheduler to check & transition.

---

## 6. Automatic Enforcements

- **T10 Fee Enforcement:** If initial response after T10 due → feesAllowed = false permanently (no override)  
- **T25 Forecast Enforcement:** If forecast > T25 due → block production until extension or petition  

---

## 7. Asset Immutability

Locked types are immutable on creation.  
Replacements create new asset with version chain link to original.

---

## 8. Audit Requirements

### 8.1 Mandatory Events

| Event | Metadata |
|-------|----------|
| CASE_CREATED | requester, channel |
| STATE_TRANSITION | fromState, toState, actor |
| ASSET_CREATED | assetId, assetType, retentionClass |
| ASSET_LOCKED | assetId |
| TIMER_COMPUTED | T10, T20, T25, tolled |
| TIMER_TOLLED | eventId, reason |
| TIMER_RESUMED | eventId, businessDaysTolled |
| FEE_WAIVED | reason, T10_due, T10_met |
| CASE_CLOSED | closureReason |
| ERROR_LOGGED | errorCode, message, stackTrace |

### 8.2 Log Integrity

Append-only. No updates. No deletes.

---

## 9. Integration Points

### 9.1 Required Integrations

| System | Purpose | Protocol |
|--------|---------|----------|
| Calendar Service | Business day computation | REST API + cache |
| Document Storage | Asset persistence | S3-compatible + SHA-256 |
| Email Gateway | Clarification, delivery | SMTP + receipts |
| Payment Processor | Fee collection | Webhook confirmation |
| Notification Service | Alerts & updates | Event-driven |

### 9.2 Webhook Events

- case.created  
- case.stateChanged  
- case.closed  
- deadline.approaching  
- deadline.missed  
- payment.received  
- payment.timeout  
- clarification.received  
- clarification.timeout  

---

## 10. Compliance Checklist

Before deployment, confirm:  

- All 22 states implemented with correct transitions  
- All transition guards enforced  
- Business day calendar loaded with MA holidays  
- Municipality closures configurable  
- T10 fee enforcement automatic and logged  
- T25 forecast enforcement automatic  
- Tolling protocol tested: start, resume, recompute  
- All locked asset types immutable on creation  
- Versioned replacement working for locked assets  
- Audit log append-only, no updates/deletes  
- All mandatory log events captured  
- Timeout scheduler configured (daily recommended)  
- Error handling for all ERR codes  
- Webhook events firing correctly  
