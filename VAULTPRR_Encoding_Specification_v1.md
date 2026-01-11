# VAULTPRRâ„¢ Encoding Specification

**Implementation Contract for Runtime Partners**  
**Version:** 1.0  
**Authority:** PublicLogicâ„¢  
**Governing Law:** M.G.L. c. 66, §10 • 950 CMR 32.00

---

## Document Purpose

This specification is the **encoding contract** between PublicLogic (governance authority) and Polimorphic (runtime executor).

It defines:
- Executable state machine with enumerated states and transitions
- Typed data schemas with validation rules
- Timer computation algorithms
- Tolling protocol with pause/resume mechanics
- Error and timeout handling
- Audit log schema

**Compliance Rule:** Polimorphic builds against this spec. PublicLogic validates against the governance architecture. Deviations require written approval.

---

## 1. State Machine

### 1.1 Enumerated States

| StateID | State Name | Type | Description |
|---------|------------|------|-------------|
| `S000` | `CREATED` | System | CASE instantiated, not yet received |
| `S100` | `INTAKE` | Form | Requester submits PRR |
| `S200` | `TIMER_COMPUTE` | System | Deadlines calculated |
| `S300` | `ASSESSMENT` | Review | RAO evaluates scope and feasibility |
| `S310` | `CLARIFICATION_SENT` | Email | Clarification request dispatched |
| `S320` | `CLARIFICATION_WAIT` | Wait | Clock tolled, awaiting response |
| `S330` | `FORECAST_REVIEW` | Decision | T25 forecast evaluation |
| `S340` | `EXTENSION_AGREEMENT` | Document | Requester agrees to extended timeline |
| `S350` | `PETITION_FILED` | Document | Petition submitted to Supervisor |
| `S400` | `GATHER` | Manual | Records search and collection |
| `S500` | `EXEMPTION_REVIEW` | Review | Exemption and segregability analysis |
| `S510` | `REDACTION` | Manual | Redactions applied |
| `S520` | `COUNSEL_REVIEW` | Gate | Legal review (optional) |
| `S600` | `PACKAGE` | Document | Response package assembled |
| `S700` | `FEE_ASSESSMENT` | Decision | Fee applicability check |
| `S710` | `INVOICE` | Document | Fee estimate and invoice |
| `S720` | `PAYMENT_WAIT` | Wait | Awaiting payment |
| `S800` | `DELIVERY` | Manual | Records transmitted |
| `S900` | `CLOSED` | Terminal | CASE complete |
| `S910` | `CLOSED_NO_RECORDS` | Terminal | No responsive records |
| `S920` | `CLOSED_WITHHELD` | Terminal | Full withholding |
| `S930` | `CLOSED_NON_PAYMENT` | Terminal | Requester did not pay |
| `S990` | `ERROR` | Terminal | System failure |
| `S991` | `TIMEOUT` | Terminal | Exceeded wait threshold |

### 1.2 Valid Transitions

```
TRANSITION_MAP = {
  S000 → S100,
  S100 → S200,
  S200 → S300,
  S300 → S310 | S330 | S400 | S910,
  S310 → S320,
  S320 → S300 | S991,
  S330 → S340 | S350 | S400,
  S340 → S400,
  S350 → S400,
  S400 → S500,
  S500 → S510 | S600 | S920,
  S510 → S520 | S600,
  S520 → S510 | S600,
  S600 → S700,
  S700 → S710 | S800,
  S710 → S720,
  S720 → S800 | S930 | S991,
  S800 → S900,
  ANY → S990 (on system error)
}
```

### 1.3 Transition Guards

| Transition | Guard Condition | Enforcement |
|------------|-----------------|-------------|
| S300 → S310 | `scope.clarificationNeeded == true` | Block if scope is determinable |
| S300 → S910 | `assessment.responsiveRecords == false` | Requires RAO attestation |
| S320 → S300 | `clarification.responseReceived == true` | Must have response payload |
| S320 → S991 | `elapsed(S320.entryTime) > CLARIFICATION_TIMEOUT` | Default: 30 calendar days |
| S330 → S340 | `forecast.completionDate > T25 AND requester.agreesToExtension == true` | Requires signed agreement asset |
| S330 → S350 | `forecast.completionDate > T25 AND requester.agreesToExtension == false` | Petition asset required |
| S500 → S920 | `exemptionReview.allExempt == true` | Withholding letter required |
| S520 → S510 | `counselReview.revisionsRequired == true` | Revision notes attached |
| S700 → S710 | `T10.met == true AND fees.required == true` | Fee estimate > $0 |
| S700 → S800 | `T10.met == false OR fees.required == false` | Fees waived or not applicable |
| S720 → S930 | `elapsed(S720.entryTime) > PAYMENT_TIMEOUT` | Default: 30 calendar days |

---

## 2. Data Schemas

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
          "enum": ["EMAIL", "MAIL", "FAX", "IN_PERSON"],
          "description": "Preferred response delivery method"
        },
        "isAnonymous": {
          "type": "boolean",
          "default": false,
          "description": "MA PRR allows anonymous requests"
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
          "format": "date-time",
          "description": "Actual receipt timestamp (UTC)"
        },
        "effectiveReceiptDate": {
          "type": "string",
          "format": "date",
          "description": "Computed start date for timers"
        },
        "channel": {
          "type": "string",
          "enum": ["WEB_FORM", "EMAIL", "MAIL", "FAX", "IN_PERSON", "PHONE"],
          "description": "How request was received"
        },
        "rawRequestText": {
          "type": "string",
          "minLength": 1,
          "description": "Verbatim request as submitted"
        },
        "attachments": {
          "type": "array",
          "items": {
            "type": "string",
            "format": "uuid",
            "description": "AssetID references"
          }
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
          "minimum": 1,
          "description": "Incremented on each clarification"
        },
        "description": {
          "type": "string",
          "minLength": 1,
          "description": "Normalized scope statement"
        },
        "dateRangeStart": {
          "type": "string",
          "format": "date"
        },
        "dateRangeEnd": {
          "type": "string",
          "format": "date"
        },
        "departments": {
          "type": "array",
          "items": { "type": "string" }
        },
        "keywords": {
          "type": "array",
          "items": { "type": "string" }
        },
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
        "effectiveReceiptDate": {
          "type": "string",
          "format": "date"
        },
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
        "eventId": {
          "type": "string",
          "format": "uuid"
        },
        "reason": {
          "type": "string",
          "enum": ["CLARIFICATION", "REQUESTER_DELAY", "SUPERVISOR_EXTENSION"]
        },
        "startTimestamp": {
          "type": "string",
          "format": "date-time"
        },
        "endTimestamp": {
          "type": "string",
          "format": "date-time"
        },
        "status": {
          "type": "string",
          "enum": ["ACTIVE", "CLOSED"]
        },
        "businessDaysTolled": {
          "type": "integer",
          "minimum": 0
        },
        "triggerAssetId": {
          "type": "string",
          "format": "uuid",
          "description": "Asset that triggered tolling (e.g., clarification email)"
        },
        "resumeAssetId": {
          "type": "string",
          "format": "uuid",
          "description": "Asset that resumed clock (e.g., requester response)"
        }
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
        "assetId": {
          "type": "string",
          "format": "uuid"
        },
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
          "enum": ["KEEPER", "REFERENCE", "TRANSACTIONAL"],
          "description": "KEEPER=permanent, REFERENCE=7yr, TRANSACTIONAL=1-2yr"
        },
        "locked": {
          "type": "boolean",
          "default": false,
          "description": "Immutable after creation"
        },
        "createdTimestamp": {
          "type": "string",
          "format": "date-time"
        },
        "createdBy": {
          "type": "string",
          "description": "Actor identifier"
        },
        "versionChain": {
          "type": "array",
          "items": {
            "type": "string",
            "format": "uuid"
          },
          "description": "Previous versions (for versioned replacement)"
        },
        "fileReference": {
          "type": "string",
          "description": "Storage location URI"
        },
        "mimeType": {
          "type": "string"
        },
        "hash": {
          "type": "string",
          "description": "SHA-256 for integrity verification"
        },
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
        "exemptionCode": {
          "type": "string",
          "pattern": "^[A-Z]{2,4}-[0-9]{1,3}[A-Za-z]?$",
          "description": "e.g., PRR-26a, HIPAA-01"
        },
        "citation": {
          "type": "string",
          "description": "Full statutory citation"
        },
        "description": {
          "type": "string"
        },
        "appliedBy": {
          "type": "string"
        },
        "appliedTimestamp": {
          "type": "string",
          "format": "date-time"
        },
        "counselApproved": {
          "type": "boolean"
        }
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
        "redactionId": {
          "type": "string",
          "format": "uuid"
        },
        "exemptionCode": {
          "type": "string"
        },
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
        "currentState": {
          "type": "string",
          "pattern": "^S[0-9]{3}$"
        },
        "lastTransition": {
          "type": "string",
          "format": "date-time"
        },
        "assignedTo": {
          "type": "string",
          "description": "RAO or delegate identifier"
        },
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
        "logId": {
          "type": "string",
          "format": "uuid"
        },
        "timestamp": {
          "type": "string",
          "format": "date-time"
        },
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
        "stateFrom": {
          "type": "string",
          "pattern": "^S[0-9]{3}$"
        },
        "stateTo": {
          "type": "string",
          "pattern": "^S[0-9]{3}$"
        },
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

```python
# CALENDAR CONFIGURATION
# Must be loaded at runtime from authoritative source

MASSACHUSETTS_LEGAL_HOLIDAYS = [
    "NEW_YEARS_DAY",           # January 1 (or observed)
    "MLK_DAY",                 # Third Monday, January
    "PRESIDENTS_DAY",          # Third Monday, February
    "PATRIOTS_DAY",            # Third Monday, April
    "MEMORIAL_DAY",            # Last Monday, May
    "JUNETEENTH",              # June 19 (or observed)
    "INDEPENDENCE_DAY",        # July 4 (or observed)
    "LABOR_DAY",               # First Monday, September
    "COLUMBUS_DAY",            # Second Monday, October
    "VETERANS_DAY",            # November 11 (or observed)
    "THANKSGIVING",            # Fourth Thursday, November
    "CHRISTMAS"                # December 25 (or observed)
]

# Municipality-specific closures loaded from configuration
# e.g., town meeting days, emergency closures
```

### 3.2 Effective Receipt Date Algorithm

```python
def compute_effective_receipt_date(received_timestamp: datetime, 
                                    municipality_calendar: Calendar) -> date:
    """
    Per M.G.L. c. 66, §10: If received on non-business day,
    effective receipt is next business day.
    """
    received_date = received_timestamp.date()
    
    if is_business_day(received_date, municipality_calendar):
        return received_date
    else:
        return next_business_day(received_date, municipality_calendar)


def is_business_day(d: date, calendar: Calendar) -> bool:
    """Returns True if d is a business day."""
    # Weekend check
    if d.weekday() >= 5:  # Saturday = 5, Sunday = 6
        return False
    
    # Holiday check
    if d in calendar.get_holidays(d.year):
        return False
    
    # Closure check (emergency, town meeting, etc.)
    if d in calendar.get_closures(d.year):
        return False
    
    return True


def next_business_day(d: date, calendar: Calendar) -> date:
    """Returns the next business day after d."""
    candidate = d + timedelta(days=1)
    while not is_business_day(candidate, calendar):
        candidate += timedelta(days=1)
    return candidate
```

### 3.3 Deadline Computation

```python
def compute_deadlines(effective_receipt_date: date,
                      municipality_calendar: Calendar) -> DeadlineModel:
    """
    Compute all statutory deadlines from effective receipt date.
    """
    return DeadlineModel(
        effectiveReceiptDate=effective_receipt_date,
        T10=compute_T10(effective_receipt_date, municipality_calendar),
        T20=compute_T20(effective_receipt_date, municipality_calendar),
        T25=compute_T25(effective_receipt_date, municipality_calendar),
        T90=compute_T90(effective_receipt_date, municipality_calendar),
        tollingEvents=[],
        businessDaysTolled=0
    )


def compute_T10(start: date, calendar: Calendar) -> T10Deadline:
    """T10: Initial written response due within 10 business days."""
    due = add_business_days(start, 10, calendar)
    return T10Deadline(dueDate=due, met=None, metTimestamp=None)


def compute_T20(start: date, calendar: Calendar) -> T20Deadline:
    """T20: Petition window closes 20 business days after receipt."""
    closes = add_business_days(start, 20, calendar)
    return T20Deadline(windowCloses=closes, petitionFiled=False)


def compute_T25(start: date, calendar: Calendar) -> T25Deadline:
    """T25: Municipal outer limit is 25 business days."""
    due = add_business_days(start, 25, calendar)
    return T25Deadline(
        dueDate=due,
        extendedDueDate=None,
        extensionGrantedBy=None
    )


def compute_T90(start: date, calendar: Calendar) -> T90Deadline:
    """T90: Appeal window is 90 calendar days from closure."""
    # Note: Computed from CLOSURE DATE, not receipt date
    # This is a placeholder; actual computation happens at closure
    return T90Deadline(windowCloses=None, appealFiled=False)


def add_business_days(start: date, days: int, calendar: Calendar) -> date:
    """Add n business days to start date."""
    current = start
    added = 0
    while added < days:
        current = current + timedelta(days=1)
        if is_business_day(current, calendar):
            added += 1
    return current
```

### 3.4 Tolling Protocol

```python
def start_tolling(case: Case, reason: TollingReason, 
                  trigger_asset_id: str) -> TollingEvent:
    """
    Pause the deadline clock.
    Called when: clarification sent, requester delay, supervisor extension.
    """
    event = TollingEvent(
        eventId=generate_uuid(),
        reason=reason,
        startTimestamp=now_utc(),
        endTimestamp=None,
        status="ACTIVE",
        businessDaysTolled=0,
        triggerAssetId=trigger_asset_id,
        resumeAssetId=None
    )
    
    case.deadlines.tollingEvents.append(event)
    
    log_action(case, "TIMER_TOLLED", metadata={
        "tollingEventId": event.eventId,
        "reason": reason
    })
    
    return event


def end_tolling(case: Case, tolling_event_id: str,
                resume_asset_id: str, calendar: Calendar) -> None:
    """
    Resume the deadline clock.
    Called when: clarification received, delay resolved.
    """
    event = find_tolling_event(case, tolling_event_id)
    
    if event.status != "ACTIVE":
        raise InvalidStateError("Tolling event not active")
    
    event.endTimestamp = now_utc()
    event.status = "CLOSED"
    event.resumeAssetId = resume_asset_id
    
    # Calculate business days tolled
    event.businessDaysTolled = count_business_days(
        event.startTimestamp.date(),
        event.endTimestamp.date(),
        calendar
    )
    
    # Update total tolled
    case.deadlines.businessDaysTolled += event.businessDaysTolled
    
    # Recompute deadlines
    recompute_deadlines(case, calendar)
    
    log_action(case, "TIMER_RESUMED", metadata={
        "tollingEventId": event.eventId,
        "businessDaysTolled": event.businessDaysTolled
    })


def recompute_deadlines(case: Case, calendar: Calendar) -> None:
    """
    Adjust all deadlines by total tolled business days.
    """
    tolled = case.deadlines.businessDaysTolled
    base = case.deadlines.effectiveReceiptDate
    
    case.deadlines.T10.dueDate = add_business_days(base, 10 + tolled, calendar)
    case.deadlines.T20.windowCloses = add_business_days(base, 20 + tolled, calendar)
    case.deadlines.T25.dueDate = add_business_days(base, 25 + tolled, calendar)
    
    log_action(case, "TIMER_COMPUTED", metadata={
        "totalTolled": tolled,
        "newT10": case.deadlines.T10.dueDate,
        "newT25": case.deadlines.T25.dueDate
    })


def count_business_days(start: date, end: date, calendar: Calendar) -> int:
    """Count business days between two dates (exclusive of end)."""
    count = 0
    current = start
    while current < end:
        current = current + timedelta(days=1)
        if is_business_day(current, calendar):
            count += 1
    return count
```

---

## 4. Validation Rules

### 4.1 Step Entry Conditions

| Step | Entry Conditions |
|------|------------------|
| `S100` INTAKE | CASE exists in S000 |
| `S200` TIMER_COMPUTE | Intake complete: requester, channel, rawRequestText populated |
| `S300` ASSESSMENT | Deadlines computed; assignedTo populated |
| `S310` CLARIFICATION_SENT | Assessment started; clarification needed flag set |
| `S400` GATHER | Assessment complete; scope.version ≥ 1; forecast ≤ T25 OR extension/petition filed |
| `S500` EXEMPTION_REVIEW | Gather complete; at least one responsive record OR search log documenting none |
| `S600` PACKAGE | Exemption review complete; all redactions applied OR no redactions needed |
| `S700` FEE_ASSESSMENT | Package created and locked |
| `S800` DELIVERY | Fee assessment complete; payment received OR fees waived/not required |
| `S900` CLOSED | Delivery confirmed |

### 4.2 Step Exit Conditions

| Step | Exit Conditions |
|------|-----------------|
| `S100` INTAKE | All required fields populated; at least one asset (original request) |
| `S200` TIMER_COMPUTE | T10, T20, T25 due dates computed and stored |
| `S300` ASSESSMENT | Decision recorded: proceed / clarify / no records |
| `S320` CLARIFICATION_WAIT | Response received OR timeout threshold reached |
| `S400` GATHER | Search log created; responsive records identified (or confirmed none) |
| `S500` EXEMPTION_REVIEW | Exemption determination made for all records |
| `S600` PACKAGE | Response package asset created and LOCKED |
| `S710` INVOICE | Invoice asset created; total calculated |
| `S720` PAYMENT_WAIT | Payment received OR timeout threshold reached |
| `S800` DELIVERY | Delivery confirmation asset created |

### 4.3 Asset Validation

| Asset Type | Required Fields | Lock Behavior |
|------------|-----------------|---------------|
| ORIGINAL_REQUEST | fileReference, hash | LOCKED on creation |
| NO_RECORDS_LETTER | fileReference, hash | LOCKED on creation |
| WITHHOLDING_LETTER | fileReference, hash, exemptions[] | LOCKED on creation |
| RESPONSE_PACKAGE | fileReference, hash, exemptions[], redactions[] | LOCKED on creation |
| EXTENSION_AGREEMENT | fileReference, hash | LOCKED on creation |
| PETITION | fileReference, hash | LOCKED on creation |
| INVOICE | total, lineItems[] | Mutable until paid |
| PAYMENT_RECEIPT | amount, paymentMethod, transactionId | LOCKED on creation |

---

## 5. Error and Timeout Handling

### 5.1 Error States

| Error Code | Trigger | Recovery |
|------------|---------|----------|
| `ERR_001` | Database write failure | Retry 3x, then S990 with manual intervention flag |
| `ERR_002` | Asset storage failure | Retry 3x, then S990 with manual intervention flag |
| `ERR_003` | Calendar service unavailable | Use cached calendar; flag for review |
| `ERR_004` | Invalid state transition attempted | Reject transition; log; alert |
| `ERR_005` | Required field missing on step exit | Block transition; return validation errors |
| `ERR_006` | Asset hash mismatch | Block; flag for integrity investigation |

### 5.2 Timeout Thresholds

| Wait State | Default Timeout | Outcome |
|------------|-----------------|---------|
| `S320` CLARIFICATION_WAIT | 30 calendar days | Transition to S991; log; notify RAO |
| `S720` PAYMENT_WAIT | 30 calendar days | Transition to S930 (CLOSED_NON_PAYMENT) |

### 5.3 Timeout Handling Protocol

```python
def check_timeouts(case: Case) -> None:
    """
    Called by scheduler (recommended: daily).
    Checks for timeout conditions and handles appropriately.
    """
    current_state = case.status.currentState
    
    if current_state == "S320":  # Clarification wait
        entry_time = get_state_entry_time(case, "S320")
        elapsed = (now_utc() - entry_time).days
        
        if elapsed >= CLARIFICATION_TIMEOUT_DAYS:
            handle_clarification_timeout(case)
    
    elif current_state == "S720":  # Payment wait
        entry_time = get_state_entry_time(case, "S720")
        elapsed = (now_utc() - entry_time).days
        
        if elapsed >= PAYMENT_TIMEOUT_DAYS:
            handle_payment_timeout(case)


def handle_clarification_timeout(case: Case) -> None:
    """Handle clarification timeout - requester non-responsive."""
    # Close any active tolling
    for event in case.deadlines.tollingEvents:
        if event.status == "ACTIVE":
            event.endTimestamp = now_utc()
            event.status = "CLOSED"
    
    # Transition to timeout state
    transition_state(case, "S991")
    
    # Log and notify
    log_action(case, "CASE_CLOSED", metadata={
        "reason": "CLARIFICATION_TIMEOUT",
        "message": "Requester did not respond to clarification within threshold"
    })
    
    notify_rao(case, "CLARIFICATION_TIMEOUT")


def handle_payment_timeout(case: Case) -> None:
    """Handle payment timeout - requester did not pay."""
    transition_state(case, "S930")
    
    log_action(case, "CASE_CLOSED", metadata={
        "reason": "NON_PAYMENT",
        "message": "Payment not received within threshold"
    })
    
    notify_rao(case, "PAYMENT_TIMEOUT")
```

---

## 6. Automatic Enforcement Rules

### 6.1 T10 Fee Enforcement

```python
def enforce_T10_fee_rule(case: Case) -> None:
    """
    Per M.G.L. c. 66, §10: Fees may not be charged if T10 missed.
    Called automatically when entering S700 (FEE_ASSESSMENT).
    """
    T10_due = case.deadlines.T10.dueDate
    T10_met_timestamp = case.deadlines.T10.metTimestamp
    
    if T10_met_timestamp is None:
        # No response recorded yet - should not reach fee assessment
        raise InvalidStateError("T10 not evaluated before fee assessment")
    
    if T10_met_timestamp.date() > T10_due:
        # T10 was missed - fees permanently waived
        case.feesAllowed = False
        
        log_action(case, "FEE_WAIVED", metadata={
            "reason": "T10_MISSED",
            "T10_due": T10_due,
            "T10_met": T10_met_timestamp.date()
        })
```

### 6.2 T25 Forecast Enforcement

```python
def enforce_T25_forecast_rule(case: Case, forecast_completion: date) -> T25Action:
    """
    Per M.G.L. c. 66, §10: Production may not exceed 25 business days unless
    requester agrees or extension granted by Supervisor.
    Called during assessment when forecasting completion.
    """
    T25_due = case.deadlines.T25.dueDate
    
    if forecast_completion <= T25_due:
        return T25Action.PROCEED
    
    # Forecast exceeds T25 - must get agreement or file petition
    return T25Action.REQUIRE_EXTENSION


class T25Action(Enum):
    PROCEED = "proceed"
    REQUIRE_EXTENSION = "require_extension"
```

---

## 7. Asset Immutability Protocol

### 7.1 Lock Enforcement

```python
def create_asset(case: Case, asset_type: AssetType, 
                 file_reference: str, **kwargs) -> Asset:
    """
    Create an asset and enforce lock rules.
    """
    asset = Asset(
        assetId=generate_uuid(),
        assetType=asset_type,
        retentionClass=get_retention_class(asset_type),
        locked=is_auto_locked(asset_type),
        createdTimestamp=now_utc(),
        createdBy=get_current_actor(),
        fileReference=file_reference,
        hash=compute_sha256(file_reference),
        **kwargs
    )
    
    case.assets.append(asset)
    
    log_action(case, "ASSET_CREATED", assetsAffected=[asset.assetId])
    
    if asset.locked:
        log_action(case, "ASSET_LOCKED", assetsAffected=[asset.assetId])
    
    return asset


def is_auto_locked(asset_type: AssetType) -> bool:
    """Assets that are immutable on creation."""
    return asset_type in [
        "ORIGINAL_REQUEST",
        "NO_RECORDS_LETTER",
        "WITHHOLDING_LETTER",
        "RESPONSE_PACKAGE",
        "EXTENSION_AGREEMENT",
        "PETITION",
        "PAYMENT_RECEIPT",
        "DELIVERY_CONFIRMATION"
    ]


def get_retention_class(asset_type: AssetType) -> str:
    """Determine retention class by asset type."""
    KEEPER_TYPES = [
        "ORIGINAL_REQUEST",
        "SCOPE_DOCUMENT",
        "SEARCH_LOG",
        "EXEMPTION_LOG",
        "NO_RECORDS_LETTER",
        "WITHHOLDING_LETTER",
        "RESPONSE_PACKAGE",
        "EXTENSION_AGREEMENT",
        "PETITION",
        "DELIVERY_CONFIRMATION"
    ]
    
    REFERENCE_TYPES = [
        "CLARIFICATION_REQUEST",
        "CLARIFICATION_RESPONSE",
        "RESPONSIVE_RECORD",
        "REDACTED_RECORD"
    ]
    
    TRANSACTIONAL_TYPES = [
        "FEE_ESTIMATE",
        "INVOICE",
        "PAYMENT_RECEIPT"
    ]
    
    if asset_type in KEEPER_TYPES:
        return "KEEPER"
    elif asset_type in REFERENCE_TYPES:
        return "REFERENCE"
    elif asset_type in TRANSACTIONAL_TYPES:
        return "TRANSACTIONAL"
    else:
        return "REFERENCE"  # Default to 7-year
```

### 7.2 Versioned Replacement

```python
def replace_locked_asset(case: Case, original_asset_id: str,
                         new_file_reference: str, reason: str) -> Asset:
    """
    Create a new version of a locked asset.
    Original is preserved; new asset links to version chain.
    """
    original = find_asset(case, original_asset_id)
    
    if not original.locked:
        raise InvalidOperationError("Use update for unlocked assets")
    
    # Build version chain
    version_chain = original.versionChain.copy() if original.versionChain else []
    version_chain.append(original.assetId)
    
    # Create replacement
    replacement = Asset(
        assetId=generate_uuid(),
        assetType=original.assetType,
        retentionClass=original.retentionClass,
        locked=True,
        createdTimestamp=now_utc(),
        createdBy=get_current_actor(),
        fileReference=new_file_reference,
        hash=compute_sha256(new_file_reference),
        versionChain=version_chain
    )
    
    case.assets.append(replacement)
    
    log_action(case, "ASSET_CREATED", 
               assetsAffected=[replacement.assetId],
               metadata={
                   "replacesAssetId": original.assetId,
                   "reason": reason,
                   "versionNumber": len(version_chain) + 1
               })
    
    return replacement
```

---

## 8. Audit Requirements

### 8.1 Mandatory Log Events

Every state transition, asset creation, and enforcement action MUST be logged.

| Event | Required Metadata |
|-------|-------------------|
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

```python
def log_action(case: Case, action: str, 
               assetsAffected: List[str] = None,
               metadata: dict = None) -> LogEntry:
    """
    Append to immutable audit log.
    Log entries cannot be modified or deleted.
    """
    entry = LogEntry(
        logId=generate_uuid(),
        timestamp=now_utc(),
        actor=get_current_actor_info(),
        action=action,
        stateFrom=case.status.currentState,
        stateTo=case.status.currentState,  # Updated by transition if applicable
        assetsAffected=assetsAffected or [],
        metadata=metadata or {}
    )
    
    case.auditLog.append(entry)
    
    # Log entries are append-only
    # No update or delete operations permitted
    
    return entry
```

---

## 9. Integration Points

### 9.1 Required Integrations

| System | Purpose | Protocol |
|--------|---------|----------|
| Calendar Service | Business day computation | REST API with cached fallback |
| Document Storage | Asset persistence | S3-compatible with SHA-256 verification |
| Email Gateway | Clarification, delivery | SMTP with delivery confirmation |
| Payment Processor | Fee collection | Webhook for payment confirmation |
| Notification Service | RAO alerts, requester updates | Event-driven |

### 9.2 Webhook Events

```json
{
  "webhookEvents": [
    "case.created",
    "case.stateChanged",
    "case.closed",
    "deadline.approaching",
    "deadline.missed",
    "payment.received",
    "payment.timeout",
    "clarification.received",
    "clarification.timeout"
  ]
}
```

---

## 10. Compliance Checklist

Before deployment, verify:

- [ ] All 22 states implemented with correct transitions
- [ ] All transition guards enforced
- [ ] Business day calendar loaded with MA holidays
- [ ] Municipality closures configurable
- [ ] T10 fee enforcement automatic and logged
- [ ] T25 forecast enforcement automatic
- [ ] Tolling protocol tested: start, resume, recompute
- [ ] All LOCKED asset types immutable on creation
- [ ] Versioned replacement working for locked assets
- [ ] Audit log append-only, no updates/deletes
- [ ] All mandatory log events captured
- [ ] Timeout scheduler configured (daily recommended)
- [ ] Error handling for all ERR codes
- [ ] Webhook events firing correctly

---

## Document Control

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2026-01-10 | PublicLogic | Initial release |

**Classification:** Confidential — PublicLogic / Polimorphic use only  
**Next Review:** Upon implementation completion or material law change

---

*This specification is the authoritative encoding contract. Deviations require written approval from PublicLogic governance authority.*
