# VAULT FISCAL™ — Governance Architecture & Domain Model

**Framework-as-a-Standard™ • Version 1.0**  
**Authority:** PublicLogic™  
**Governing Law:** M.G.L. c. 41, §23 (Municipal Finance); M.G.L. c. 149 (Procurement)

---

## Document Classification

| Attribute | Value |
|-----------|-------|
| **Type** | Governance Architecture |
| **Status** | Authoritative |
| **Confidentiality** | Internal / NDA Partners |
| **Owner** | PublicLogic™ |
| **Encoding Spec** | VAULT_FISCAL_Encoding_Specification_v1.md |

---

## Purpose

VAULT FISCAL™ defines the **governed, enforcement-first operating system** for Massachusetts municipal financial processes including procurement, invoice processing, and payment.

This document converts procurement and finance statutes into **structural requirements** so that:

1. **Invoices are matched to purchase orders and receipts** — 3-way match is enforced before payment
2. **Approvals are documented and locked** — authorization trail is unbreakable
3. **Budget constraints are verified** — overspending is prevented
4. **Payments are auditable** — all transactions create immutable records
5. **Risk is carried by structure, not individuals** — operators cannot bypass controls

**Core Principle:** If a financial transaction isn't recordable, transferable, and auditable, it violates fiduciary duty.

---

## Part I: Statutory Authority

### Governing Law

VAULT FISCAL™ enforces financial control statutes:

| Statute | Authority | VAULT FISCAL Enforcement |
|---|---|---|
| **M.G.L. c. 41, §23** | Municipal Finance; fiscal powers of select board | Budget encumbrance, vendor management |
| **M.G.L. c. 149** | Procurement law; competitive bidding thresholds | Bid tracking, vendor authority |
| **M.G.L. c. 30B** | Uniform procurement; bidding requirements | Competitive process gates |
| **Charter/ByLaw** | Local spending authority limits | Approval matrix per signature authority |

### Core Financial Control Requirements

| Requirement | Authority | VAULT FISCAL Enforcement |
|---|---|---|
| **3-Way Match** | Best accounting practice; state auditor standard | InvoiceAmount = POAmount = ReceiptAmount (gate enforced) |
| **Approval Authority** | Charter/ByLaw; spending thresholds | Approval chain varies by amount; signatures required |
| **Budget Availability** | M.G.L. c. 41, §23 | Cannot exceed available, unencumbered funds |
| **Vendor Authorization** | Charter/ByLaw | Only approved vendors can be paid |
| **Segregation of Duties** | Best practice; audit requirement | Different actors for PO creation, receipt, approval, payment |
| **Immutable Payment Records** | GASB/FUND accounting standards | Payment records locked; cannot be edited after posting |

---

## Part II: Enforcement Architecture

### CASE Lifecycle for Invoices

```
INTAKE
  ↓ [Invoice received; basic validation]
MATCHING
  ↓ [Match invoice to PO; match to receipt]
VARIANCE_REVIEW (if 3-way match fails)
  ↓ [Investigate amount differences]
APPROVAL
  ↓ [Route to appropriate approval authority per amount]
PAYMENT_APPROVAL
  ↓ [Final approval; payment authorized]
PAYMENT_PROCESSING
  ↓ [Check/ACH/card generated and sent]
RECONCILIATION
  ↓ [Payment reconciled against ledger]
CLOSED
```

### Financial Control Gates

#### Gate 1: 3-Way Match Verification

**Condition:** InvoiceAmount = POAmount = ReceiptAmount

| Result | Action |
|---|---|
| **MATCH** | Proceed to approval |
| **VARIANCE** | Flag variance; request explanation; block payment until resolved |
| **NO_PO** | Block payment; escalate to procurement (PO should have been created first) |
| **NO_RECEIPT** | Block payment; escalate to warehouse/receiving (goods must be received) |

---

#### Gate 2: Budget Availability Check

**Condition:** Unencumbered budget available >= InvoiceAmount

| Result | Action |
|---|---|
| **AVAILABLE** | Proceed to approval |
| **INSUFFICIENT** | Block payment; flag for budget review |
| **EXPIRED_FISCAL_YEAR** | Block payment; escalate (different rules for year-end) |

---

#### Gate 3: Approval Authority

**Condition:** Approval routed to correct authority per amount threshold

| Amount | Authority | Signature | Override Allowed? |
|---|---|---|---|
| $0 - $1,000 | Department Head | Yes | No |
| $1,001 - $5,000 | Finance Director | Yes | No |
| $5,001 - $25,000 | Treasurer | Yes | No |
| $25,001+ | Town Meeting / BOS | Yes | No |

---

#### Gate 4: Vendor Authorization

**Condition:** Vendor is approved and not debarred

| Result | Action |
|---|---|
| **APPROVED** | Proceed to payment |
| **NOT_IN_SYSTEM** | Block; create vendor record (or use existing) |
| **DEBARRED** | Block; escalate to legal |
| **EXPIRED** | Block; renew vendor status |

---

#### Gate 5: Duplicate Payment Check

**Condition:** Verify invoice not already paid

| Result | Action |
|---|---|
| **NOT_PAID** | Proceed to payment |
| **ALREADY_PAID** | Block; investigate (duplicate submission?) |
| **PARTIAL_PAID** | Flag; confirm amount is remaining due |

---

### Payment Processing Flow

```
Approved Invoice
  ↓
Payment Method Selected (Check, ACH, Card)
  ↓
Payment Batch Processed
  ├─ Check: Print, sign, mail
  ├─ ACH: Transmit to bank
  └─ Card: Process through gateway
  ↓
Payment Confirmation Received
  ↓
GL Posting Created (LOCKED)
  ↓
Reconciliation (Monthly: verify payment cleared bank)
  ↓
CLOSED
```

---

### Timer Model

| Timer | Definition | Consequence |
|---|---|---|
| **T_PAYMENT_DUE** | Payment deadline per vendor terms (e.g., Net 30) | If missed: Late payment fee or vendor dispute |
| **T_APPROVAL_WINDOW** | Days to complete approval before payment is blocked | If exceeded: Flag for expediting |
| **T_RECONCILIATION** | Days to reconcile payment against bank statement | If exceeded: Flag variance |

**Note:** Unlike PRR or CLERK, FISCAL timers are typically soft (targets) rather than hard (enforcement). However, payment deadlines are tracked for vendor relationship management and audit purposes.

---

### Segregation of Duties (Structural)

VAULT FISCAL enforces role separation:

| Role | Responsibility | Cannot Do |
|---|---|---|
| **Requester** | Create purchase requisition | Approve or pay |
| **Procurement** | Create PO; verify budget | Receive goods; approve payment |
| **Receiver** | Verify goods received; create receipt | Approve payment |
| **Approver** | Review and approve payment | Receive goods; process payment |
| **Treasurer** | Process payment | Approve (depends on amount) |
| **Accountant** | Post to GL; reconcile | Approve |

**Enforcement:** System prevents one actor from performing multiple roles on same CASE (where audit risk exists).

---

## Part III: Domain Model

### Entity Relationship Overview

```
CASE (root container)
├── Vendor (who is being paid)
├── Invoice (what they submitted)
├── PurchaseOrder (authorization to buy)
├── Receipt (proof goods were received)
├── ApprovalChain (who must approve, per amount)
├── PaymentMethod (check, ACH, card)
├── Deadlines (payment due, approval window, reconciliation)
├── Assets (all documents: invoice scan, PO, receipt, approval signatures, payment confirmation)
└── AuditLog (all actions: receipt, matching, approval, payment)
```

### Entity Definitions

#### CASE

| Attribute | Type | Description |
|---|---|---|
| CaseID | UUID | Immutable unique identifier |
| CaseType | String | Always "FISCAL" |
| VendorID | UUID | Linked to vendor master record |
| VendorName | String | Vendor name |
| InvoiceNumber | String | Vendor's invoice number |
| InvoiceDate | Date | Date invoice was created |
| ReceiptDate | Date | Date received by municipality |
| Amount | Decimal | Invoice amount |
| Currency | String | USD (default) |
| Description | String | What was purchased |
| PONumber | String | Linked purchase order |
| ReceiptNumber | String | Linked receipt/delivery |
| ApprovalAuthority | String | Who must approve (per amount threshold) |
| ApprovalStatus | Enum | PENDING, APPROVED, DENIED |
| PaymentStatus | Enum | NOT_DUE, AUTHORIZED, PROCESSING, PAID, VOID |
| MatchStatus | Enum | MATCHED, VARIANCE, UNMATCHED |
| Assets | Array | Invoice, PO, receipt, approvals, payment confirmation |
| AuditLog | Array | All decisions and actions |

#### Vendor Schema

```json
{
  "vendorId": "UUID",
  "vendorName": "MainTech Services",
  "vendorType": "Business",
  "taxId": "12-3456789",
  "status": "ACTIVE",
  "approvedDate": "2025-06-01",
  "expiryDate": "2026-06-01",
  "isDebarred": false,
  "paymentTerms": "Net 30",
  "paymentMethods": ["Check", "ACH"],
  "contactName": "Jane Doe",
  "email": "ap@maintech.com",
  "phone": "555-0100"
}
```

#### PurchaseOrder Schema

```json
{
  "poNumber": "PO-2026-0892",
  "vendorId": "UUID",
  "amount": 4500.00,
  "glCode": "4100.001",
  "department": "IT",
  "description": "Professional services",
  "dateCreated": "2026-01-15",
  "dateAuthorized": "2026-01-16",
  "status": "AUTHORIZED",
  "linkedCases": ["fiscal-2026-0892"]
}
```

#### Receipt Schema

```json
{
  "receiptNumber": "REC-2026-0145",
  "poNumber": "PO-2026-0892",
  "amount": 4500.00,
  "dateReceived": "2026-01-31",
  "receivedBy": "Alex Smith",
  "description": "Services completed through Jan 31",
  "status": "RECEIVED",
  "notes": "All deliverables accepted; no exceptions"
}
```

---

## Part IV: Critical Gates

### Gate 1: Invoice Validation

**When:** INTAKE state
**Condition:** Invoice has required fields (vendor, amount, PO reference)
**Pass:** Proceed to MATCHING
**Fail:** Request resubmission with complete information

---

### Gate 2: 3-Way Match

**When:** MATCHING state
**Condition:** InvoiceAmount == POAmount == ReceiptAmount
**MATCH:** Proceed to APPROVAL
**VARIANCE:** Flag variance; block payment until explained and resolved
**MISSING:** Block payment; escalate to procurement or receiving

---

### Gate 3: Budget Availability

**When:** APPROVAL state
**Condition:** Unencumbered budget >= InvoiceAmount
**AVAILABLE:** Proceed
**INSUFFICIENT:** Block; flag for budget review

---

### Gate 4: Approval Authority

**When:** APPROVAL state
**Condition:** Correct authority approves per amount threshold
**APPROVED:** Lock approval signature; proceed to PAYMENT_APPROVAL
**DENIED:** Lock denial; close CASE
**ESCALATE:** If amount requires higher authority, route accordingly

---

### Gate 5: Duplicate Payment Detection

**When:** PAYMENT_APPROVAL state
**Condition:** Verify invoice not previously paid
**UNIQUE:** Proceed to PAYMENT_PROCESSING
**DUPLICATE:** Block; investigate

---

## Part V: Implementation Boundaries

### PublicLogic™ Responsibility

- Framework definition and governance authority
- Statutory interpretation and financial control rules
- Domain model and entity definitions
- Process architecture and state definitions
- Segregation of duties enforcement

### Runtime Partner Responsibility (Polimorphic)

- State machine implementation
- Data schema implementation
- 3-way match validation logic
- Budget encumbrance integration
- Integration with GL/accounting system
- Payment processing integration
- User interface

### Municipal Authority Responsibility

- Vendor master data maintenance
- Approval authority designation (by amount)
- GL code assignment
- Budget availability verification
- Payment method configuration

---

## Part VI: Compliance Requirements

### Audit Trail Requirements

Every FISCAL CASE must maintain immutable audit log capturing:

- Invoice receipt and validation
- All matching activities (invoice vs. PO vs. receipt)
- Variance detection and resolution
- All approval signatures (immutable)
- Budget checking and encumbrance
- Payment authorization
- Payment processing
- GL posting
- Reconciliation

### Retention Requirements

| Record Type | Minimum Retention |
|---|---|
| CASE container | 7 years (per audit requirement) |
| Invoice | Permanent (KEEPER) |
| PO | Permanent (KEEPER) |
| Receipt | Permanent (KEEPER) |
| Approval signatures | Permanent (KEEPER) |
| Payment confirmation | Permanent (KEEPER) |
| GL posting | Permanent (KEEPER) |
| Audit log | Permanent (KEEPER) |

---

*This document is the authoritative governance architecture for VAULT FISCAL™. Implementation must conform to the companion Encoding Specification. Deviations require written approval from PublicLogic.*
