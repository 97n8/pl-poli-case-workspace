# VAULT FISCAL Module — Charter

**Accounts Payable, Procurement, Budget • Municipal Finance**

---

## Module Purpose

VAULT FISCAL enforces procurement and payment processes as CASE-based work.

Examples:
* Invoice intake and payment
* Purchase order processing
* Approval workflows
* Budget encumbrance
* Vendor management

Each invoice or PO is one CASE.

---

## Statutory Authority

**M.G.L. c. 41, §23** — Municipal Finance
**M.G.L. c. 149** — Public Procurement Law

---

## CASE Model for FISCAL (Invoice Example)

### Identity

```json
{
  "CaseID": "fiscal-2026-0892",
  "CaseType": "FISCAL",
  "SubType": "Invoice",
  "CreatedTimestamp": "2026-02-01T14:30:00Z"
}
```

### Subject

```json
{
  "VendorIdentity": {
    "VendorID": "V-001",
    "Name": "MainTech Services",
    "Address": "456 Tech Blvd"
  },
  "Amount": 4500.00,
  "Currency": "USD",
  "InvoiceNumber": "INV-2026-001",
  "PeriodCovered": "2026-01-01 to 2026-01-31"
}
```

### Scope

```json
{
  "ScopeDefinition": {
    "Description": "Professional services - IT support",
    "GLCode": "4100.001",
    "CostCenter": "IT",
    "BudgetAuthority": "Director of IT"
  }
}
```

### Deadlines

```json
{
  "PaymentDue": {
    "DueDate": "2026-03-02",
    "Description": "Net 30 per contract"
  },
  "VarianceWindow": {
    "DueDate": "2026-02-20",
    "Description": "3-way match (PO/receipt/invoice) must be resolved"
  }
}
```

### Assets

```json
{
  "Assets": [
    {"AssetType": "Invoice", "RetentionClass": "KEEPER"},
    {"AssetType": "PurchaseOrder", "RetentionClass": "KEEPER"},
    {"AssetType": "Receipt", "RetentionClass": "KEEPER"},
    {"AssetType": "ApprovalSignature", "RetentionClass": "KEEPER", "Locked": true},
    {"AssetType": "PaymentConfirmation", "RetentionClass": "TRANSACTIONAL"}
  ]
}
```

---

## Module-Specific Rules

### 3-Way Match

Before payment:
1. Invoice amount matches PO amount
2. Receipt amount matches invoice
3. All three are documented

If variance: Require approval from vendor/budget authority before payment.

### Approval Chain

Invoices may require multiple approvals:
1. Department head (correctness)
2. Accounting (coding, budget availability)
3. Finance director (payment authority)

CASE records all approvals with signatures.

### Payment Methods

Payment can be:
* Check
* ACH transfer
* Credit card
* Other per municipal policy

CASE records payment method and confirmation.

---

## Process Flow (High Level)

```
Intake → 3-Way Match → Approval Chain → Payment Processing → Reconciliation → Closure
```

---

## Asset Types in FISCAL

| Asset | Retention | Locked? | Purpose |
|-------|-----------|---------|---------|
| Invoice | KEEPER | Yes | Original invoice from vendor |
| PurchaseOrder | KEEPER | Yes | Authorization to purchase |
| Receipt | KEEPER | Yes | Evidence goods/services received |
| ApprovalSignature | KEEPER | Yes | Authorized by appropriate authority |
| PaymentConfirmation | TRANSACTIONAL | No | Proof of payment |
| Variance Report | REFERENCE | No | If amount differences, investigation |

---

## Decision Tables

### Gate 1: 3-Way Match Variance

| Invoice Amt | PO Amt | Receipt Amt | Match? | Action |
|-----|-----|-----|---|---|
| $1,000 | $1,000 | $1,000 | YES ✓ | Proceed |
| $1,000 | $900 | $1,000 | NO | Variance: invoice exceeds PO |
| $1,000 | $1,000 | $950 | NO | Variance: receipt differs |
| $1,100 | $1,000 | $1,100 | NO | Variance: invoice > PO (over-spend?) |

**Variance Resolution:** Require approval from budget authority before payment

---

### Gate 2: Budget Availability

| Amount | Budget Available | Encumbered | Can Proceed? | Action |
|---|---|---|---|---|
| $5,000 | $10,000 | $0 | YES ✓ | Proceed |
| $5,000 | $3,000 | $0 | NO | Reject: insufficient budget |
| $5,000 | $8,000 | $5,000 | NO | Reject: insufficient unencumbered |
| $5,000 | $10,000 | $6,000 | YES ✓ | Proceed (if $5K available) |

---

### Approval Matrix

| Amount | Department | Authority | Signature |
|---|---|---|---|
| $0 - $1,000 | Any | Department Head | Required |
| $1,001 - $5,000 | Any | Finance Director | Required |
| $5,001 - $25,000 | Any | Treasurer | Required |
| $25,001+ | Any | Board of Selectmen | Required |

---

## Process Flow (High-Level)

```
[Invoice Intake]
  ↓
[Locate PO] → [Match PO to Invoice]
  ↓
[Locate Receipt] → [Match Receipt to Invoice]
  ↓
[3-Way Match Pass?]
  ├─ NO → [Log Variance] → [Request Approval] → [Recheck]
  └─ YES → [Check Budget Available]
  ↓
[Budget Available?]
  ├─ NO → [Reject: insufficient budget]
  └─ YES → [Route to Approval Authority per amount]
  ↓
[Approval Signature] → [LOCKED]
  ↓
[Calculate Payment Method] (check, ACH, card)
  ↓
[Process Payment] → [Confirmation Record]
  ↓
[Post to GL] → [Reconciliation] → [LOCKED]
  ↓
[CLOSED]
```

---

## Compliance Checklist for FISCAL CASE

Before closing:

- [ ] Invoice receipt recorded
- [ ] PO located and matched
- [ ] Receipt confirmed (goods/services delivered)
- [ ] 3-way match verified (or variance documented and approved)
- [ ] Budget availability checked
- [ ] Approval chain completed (per amount thresholds)
- [ ] Payment processed with confirmation
- [ ] GL posting completed and verified
- [ ] Reconciliation completed
- [ ] Key assets locked (approval, payment confirmation)
- [ ] Audit log sealed
- [ ] No transition blockers remain

---

## Module Expansion Notes

Detailed implementation will include:
* Vendor master data integration
* Budget interface (if separate system)
* Multi-currency support (if needed)
* Recurring invoice handling
* Debit memo / credit memo process
* Audit hold / legal hold capability

---

*VAULT FISCAL follows the same CASE architecture as PRR. Finance-specific rules (3-way match, approval chain) are module-specific.*
