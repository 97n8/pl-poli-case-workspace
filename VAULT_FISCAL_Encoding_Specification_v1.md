# VAULT FISCAL™ — Encoding Specification

**Implementation Contract • Version 1.0**  
**Authority:** PublicLogic™  
**For Runtime Partners:** Polimorphic

---

## Part I: State Machine Definition

### FISCAL State Enum

```json
{
  "states": [
    "INTAKE",
    "VALIDATION",
    "MATCHING",
    "VARIANCE_REVIEW",
    "BUDGET_CHECK",
    "APPROVAL_ROUTING",
    "APPROVAL_PENDING",
    "APPROVED",
    "PAYMENT_PROCESSING",
    "PAYMENT_COMPLETE",
    "RECONCILIATION",
    "CLOSED"
  ]
}
```

### State Transition Matrix

| From | To | Condition | Action |
|---|---|---|---|
| INTAKE | VALIDATION | Invoice received | Validate required fields |
| VALIDATION | MATCHING | All fields present | Match to PO and receipt |
| MATCHING | VARIANCE_REVIEW | Amounts don't match | Request variance explanation |
| MATCHING | BUDGET_CHECK | Amounts match | Check budget availability |
| VARIANCE_REVIEW | BUDGET_CHECK | Variance resolved | Proceed |
| VARIANCE_REVIEW | INTAKE | Unresolved variance | Request resubmission |
| BUDGET_CHECK | APPROVAL_ROUTING | Budget available | Route to approval authority |
| BUDGET_CHECK | INTAKE | Budget insufficient | Escalate for budget decision |
| APPROVAL_ROUTING | APPROVAL_PENDING | Correct authority assigned | Wait for approval |
| APPROVAL_PENDING | APPROVED | Authority approves | Lock approval signature |
| APPROVAL_PENDING | INTAKE | Authority denies | Request resubmission |
| APPROVED | PAYMENT_PROCESSING | Payment scheduled | Generate check/ACH |
| PAYMENT_PROCESSING | PAYMENT_COMPLETE | Payment sent | Record confirmation |
| PAYMENT_COMPLETE | RECONCILIATION | Bank confirmation received | Reconcile to GL |
| RECONCILIATION | CLOSED | All cleared | Lock GL posting |

---

## Part II: JSON Schemas

### Invoice CASE Schema

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "FISCAL Invoice CASE",
  "type": "object",
  "properties": {
    "caseId": { "type": "string" },
    "vendor": {
      "type": "object",
      "properties": {
        "vendorId": { "type": "string" },
        "status": { "type": "string", "enum": ["ACTIVE", "INACTIVE", "DEBARRED"] }
      }
    },
    "invoice": {
      "type": "object",
      "properties": {
        "invoiceNumber": { "type": "string" },
        "amount": { "type": "number", "minimum": 0 }
      }
    },
    "matching": {
      "type": "object",
      "properties": {
        "poAmount": { "type": "number" },
        "receiptAmount": { "type": "number" },
        "matchStatus": { "type": "string", "enum": ["MATCHED", "VARIANCE", "UNMATCHED"] }
      }
    },
    "approval": {
      "type": "object",
      "properties": {
        "approvalStatus": { "type": "string", "enum": ["PENDING", "APPROVED", "DENIED"] },
        "approvalSignatureLocked": { "type": "boolean" }
      }
    }
  }
}
```

---

## Part III: Critical Validation Rules

### Rule: 3-Way Match Verification

```
IF InvoiceAmount = POAmount = ReceiptAmount (±0.01 tolerance):
  PASS: Proceed to budget check
ELSE:
  FAIL: Flag variance; request explanation; block payment
```

### Rule: Budget Availability

```
IF UnencumberedBudget >= InvoiceAmount:
  PASS: Proceed to approval
ELSE:
  FAIL: Block payment; escalate for budget review
```

### Rule: Approval Authority Routing

```
IF Amount <= $1,000:
  Route to Department Head
ELSIF Amount <= $5,000:
  Route to Finance Director
ELSIF Amount <= $25,000:
  Route to Treasurer
ELSE:
  Route to Board of Selectmen
```

### Rule: Approval Signatures LOCKED

```
IF ApprovalSignature recorded:
  LOCK: Cannot be edited
  IF Edit attempted:
    BLOCK: Return error E_CANNOT_EDIT_LOCKED
```

### Rule: GL Posting LOCKED

```
IF PostedToGL:
  LOCK: Cannot be reversed in-place
  IF Correction needed:
    CREATE: Reversing entry + new posting (audit trail preserved)
```

---

## Part IV: Timer Algorithms

### T_PAYMENT_DUE (Payment Deadline)

```
T_PAYMENT_DUE = InvoiceDate + VendorPaymentTerms (e.g., Net 30)

IF CurrentDate > T_PAYMENT_DUE AND PaymentStatus != PAID:
  Log: "Payment overdue"
  Notify: Treasurer
```

### T_APPROVAL_WINDOW (Approval Deadline)

```
T_APPROVAL_WINDOW = InvoiceReceivedDate + 5 business days

IF CurrentDate > T_APPROVAL_WINDOW AND ApprovalStatus = PENDING:
  Action: Escalate to next authority level
  Log: "Approval window exceeded"
```

### T_RECONCILIATION (Bank Reconciliation Deadline)

```
T_RECONCILIATION = PaymentClearedDate + 30 calendar days

IF CurrentDate > T_RECONCILIATION AND Status != RECONCILED:
  Log: "Reconciliation overdue"
  Flag: Variance investigation needed
```

---

## Part V: Error Handling

| Code | Condition | Action |
|---|---|---|
| E_3WAY_MATCH_FAILED | Amounts don't match | Request variance explanation |
| E_BUDGET_INSUFFICIENT | Insufficient funds | Escalate; block payment |
| E_VENDOR_INACTIVE | Vendor not active/debarred | Escalate to procurement |
| E_DUPLICATE_DETECTED | Invoice already paid | Investigate; escalate |
| E_APPROVAL_DENIED | Authority rejects | Request resubmission |
| E_CANNOT_EDIT_LOCKED | Edit locked approval/GL | Explain immutability; offer new version |

---

## Part VI: Deployment Checklist

- [ ] 3-way match validation tested (all scenarios)
- [ ] Budget encumbrance integration tested
- [ ] Approval routing tested for all thresholds
- [ ] Payment integration tested (check, ACH)
- [ ] GL posting integration tested
- [ ] Bank reconciliation integration tested
- [ ] Segregation of duties enforced
- [ ] Vendor debarment check integrated
- [ ] Duplicate detection tested
- [ ] All error codes tested
- [ ] Approvals locked and audit logged
- [ ] GL posting locked and audit logged

---

*VAULT FISCAL Encoding Specification v1.0. Build exactly as specified.*
