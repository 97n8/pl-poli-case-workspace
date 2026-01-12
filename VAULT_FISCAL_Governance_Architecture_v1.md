VAULT FISCAL™ — Governance Architecture & Domain Model
**Framework-as-a-Standard™ • Version 1.0**
**Authority:** PublicLogic™
**Governing Law:** M.G.L. c. 41 §56 (Warrants/Payment); M.G.L. c. 30B (Uniform Procurement); M.G.L. c. 149 (Construction Procurement)
---
## Document Classification
| Attribute       | Value                          |
|-----------------|--------------------------------|
| Type            | Governance Architecture        |
| Status          | Authoritative                  |
| Confidentiality | Internal / NDA Partners        |
| Owner           | PublicLogic™                   |
| Encoding Spec   | VAULT_FISCAL_Encoding_Specification_v1.md |

---

## Purpose
VAULT FISCAL™ operates as the **enforcement-first** system for MA municipal finance: procurement, invoice processing, payment. It translates statutes into structural invariants so:
1. 3-way match is mandatory pre-payment.
2. Approvals are locked/immutable.
3. Budget overspend is impossible.
4. Transactions are fully auditable.
5. Risk is structural, not personal.

**Core Principle:** Unrecordable/untransferable/unauditable transactions violate fiduciary duty.

---

## Part I: Statutory Authority
### Governing Statutes
| Statute                  | Key Authority                                      | VAULT FISCAL Enforcement                          |
|--------------------------|----------------------------------------------------|---------------------------------------------------|
| **M.G.L. c. 41 §56**    | Warrants for bills: approval, accountant examination, warrant draw, treasurer payment only on warrant; disallowance for fraudulent/unlawful/excessive | 3-way match (delivery verification), locked approvals, warrant-equivalent gates before payment |
| **M.G.L. c. 41 §52**    | Auditor/selectmen disallowance                     | Variance/budget rejection paths                   |
| **M.G.L. c. 30B**       | Uniform procurement (supplies/services)            | Bid/quote gates, vendor authorization             |
| **M.G.L. c. 149 §§44A–44M** | Construction bidding/progress payments          | Applicable for building-related invoices          |
| **Local Charter/Bylaw** | Spending thresholds, approval authorities          | Amount-based matrix                               |

**3-Way Match** — Best practice (auditor/DLS/GFOA standard) enforcing §56 delivery/rendering verification.

---

## Part II: Enforcement Architecture
### CASE Lifecycle for Invoices
INTAKE → MATCHING → VARIANCE_REVIEW (if mismatch) → APPROVAL → PAYMENT_APPROVAL → PAYMENT_PROCESSING → RECONCILIATION → CLOSED
text### Financial Control Gates
#### Gate 1: 3-Way Match
**Condition:** InvoiceAmount ≈ POAmount ≈ ReceiptAmount (± tolerance)
| Result     | Action                                      |
|------------|---------------------------------------------|
| MATCH     | Proceed                                     |
| VARIANCE  | Flag; block until resolved/approved         |
| NO_PO     | Block; escalate procurement                 |
| NO_RECEIPT| Block; escalate receiving                   |

#### Gate 2: Budget Availability
**Condition:** Unencumbered ≥ InvoiceAmount
| Result       | Action                          |
|--------------|---------------------------------|
| AVAILABLE   | Proceed                         |
| INSUFFICIENT| Block; escalate review          |
| EXPIRED_YEAR| Block; year-end rules           |

#### Gate 3: Approval Authority
| Amount          | Authority            | Signature | Override? |
|-----------------|----------------------|-----------|-----------|
| $0–$1,000      | Department Head      | Yes      | No        |
| $1,001–$5,000  | Finance Director     | Yes      | No        |
| $5,001–$25,000 | Treasurer            | Yes      | No        |
| $25,001+       | Board of Selectmen   | Yes      | No        |

#### Gate 4: Vendor Authorization
| Result     | Action                          |
|------------|---------------------------------|
| APPROVED  | Proceed                         |
| NOT_IN_SYSTEM | Block; create/verify record |
| DEBARRED  | Block; escalate legal           |

#### Gate 5: Duplicate Check
| Result        | Action                          |
|---------------|---------------------------------|
| UNIQUE       | Proceed                         |
| ALREADY_PAID | Block; investigate              |
| PARTIAL      | Flag; confirm balance           |

### Payment Processing Flow
Approved → Method Select → Batch Process → Confirmation → GL Post (LOCKED) → Reconcile → CLOSED

### Timer Model
Timers are soft (flags/notifications) for finance flexibility.

| Timer                | Definition                     | Consequence                     |
|----------------------|--------------------------------|---------------------------------|
| T_PAYMENT_DUE       | Vendor terms (e.g., Net 30)   | Flag overdue; notify treasurer  |
| T_APPROVAL_WINDOW   | e.g., 5–10 business days      | Escalate stalled routing        |
| T_RECONCILIATION    | e.g., 30 days post-clear      | Flag investigation              |

### Segregation of Duties
| Role         | Can Do                          | Cannot Do                       |
|--------------|---------------------------------|---------------------------------|
| Requester   | Requisition                     | Approve/pay                     |
| Procurement | PO creation                     | Receive/approve/pay             |
| Receiver    | Receipt                         | Approve/pay                     |
| Approver    | Review/approve                  | Receive/process                 |
| Treasurer   | Payment                         | Approve (per threshold)         |
| Accountant  | GL post/reconcile               | Approve                         |

Enforced via role checks per CASE action.

---

## Part III: Domain Model
### ER Overview
CASE (root)
├── Vendor
├── Invoice
├── PurchaseOrder
├── Receipt
├── ApprovalChain
├── PaymentMethod
├── Deadlines
├── Assets
└── AuditLog
text### Key Entity Definitions
#### CASE (Core)
- CaseID: UUID
- CaseType: "FISCAL"
- VendorID, InvoiceNumber, Amount, PONumber, ReceiptNumber, ApprovalStatus, PaymentStatus, MatchStatus
- Assets & AuditLog arrays

#### Vendor (Master)
```json
{
  "vendorId": "UUID",
  "name": "MainTech Services",
  "taxId": "12-3456789",
  "status": "ACTIVE",  // ACTIVE, INACTIVE, DEBARRED
  "paymentTerms": "Net 30",
  "methods": ["Check", "ACH"],
  "expiryDate": "2026-06-01"
}
PurchaseOrder & Receipt — Similar structured schemas (as in original).

Part IV: Implementation Boundaries

PublicLogic™: Governance, statutes, domain, controls.
Polimorphic: State machine, schemas, gates, UI, SSI integrations (GL/budget/payment).
Municipality: Vendor/budget data, authority designations.


Part V: Compliance Requirements

Audit Trail: Immutable log of all gates/actions.
Retention: KEEPER (permanent/7+ years) for core fiscal records.

Authoritative governance for VAULT FISCAL™. Conform to Encoding Spec v1. Deviations require PublicLogic approval.
