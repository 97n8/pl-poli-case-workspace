text# VAULT FISCAL Module — Charter
**Accounts Payable • Procurement • Budget Encumbrance • Municipal Finance**

---

## Module Purpose

VAULT FISCAL enforces strict, auditable procurement and payment processes using the CASE-based architecture. Every financial work unit is encapsulated as a single, immutable CASE.

**Core Examples**:
- Invoice intake, 3-way match, approval, and payment
- Purchase order creation and issuance
- Multi-level approval workflows
- Budget availability checks and encumbrance
- Vendor onboarding and management

**Principle**: One invoice = one CASE; one PO = one CASE. No external documents or decisions exist outside the CASE.

---

## Statutory Authority & Compliance Foundation

- **M.G.L. c. 41 §56** — Warrants for payment of bills (primary): Requires department/board approval after verifying charges are correct, goods/services ordered/delivered/rendered; town accountant examines and draws warrant; treasurer pays only on approved warrant; disallowance for fraudulent/unlawful/excessive claims.
- **M.G.L. c. 41 §52** — Approval of bills (supplemental): Auditor/selectmen may disallow claims and document reasons.
- **M.G.L. c. 149 §§44A–44M** — Public construction procurement (when applicable): Governs bidding for building-related work; intersects with AP for progress payments/invoices.
- **M.G.L. c. 30 §39M** — Public works bidding (supplies/services over certain thresholds).
- **Best Practices**: 3-way match (PO + Invoice + Receipt) exceeds statutory minimums but aligns with MA DLS/auditor recommendations and GFOA standards for fraud prevention and fiscal control.

VAULT FISCAL enforces these via gates, locked assets, and transition blockers — ensuring warrant-like approval before payment.

---

## CASE Model — Invoice Example

### 1. Identity (Immutable)

```json
{
  "CaseID": "fiscal-2026-0892",
  "CaseType": "FISCAL",
  "SubType": "Invoice",
  "CreatedTimestamp": "2026-02-01T14:30:00Z",
  "CreatedBy": "accounting@lawrence.ma.us"
}
2. Subject (Sufficient for quick understanding)
JSON{
  "VendorIdentity": {
    "VendorID": "V-001",
    "Name": "MainTech Services",
    "Address": "456 Tech Blvd, Anytown, MA"
  },
  "Amount": 4500.00,
  "Currency": "USD",
  "InvoiceNumber": "INV-2026-001",
  "PeriodCovered": "2026-01-01 to 2026-01-31"
}
3. Scope (Versioned)
JSON{
  "ScopeDefinition": {
    "Description": "Professional IT support services per contract",
    "GLCode": "4100.001",
    "CostCenter": "IT Department",
    "BudgetAuthority": "Director of IT"
  },
  "ScopeVersion": 1
}
4. Deadlines (Enforced with Consequences)
JSON{
  "PaymentDue": {
    "DueDate": "2026-03-02",
    "Description": "Net 30 per contract; flag late payment interest if applicable"
  },
  "VarianceResolutionWindow": {
    "DueDate": "2026-02-20",
    "Description": "Resolve 3-way match variances; blocker if missed"
  },
  "ApprovalRoutingDeadline": {
    "DueDate": "2026-02-15",
    "Description": "Escalate stalled approvals per policy"
  }
}
5. Assets (All Inside CASE)
JSON{
  "Assets": [
    {"AssetType": "Invoice", "RetentionClass": "KEEPER", "Locked": true},
    {"AssetType": "PurchaseOrder", "RetentionClass": "KEEPER", "Locked": true},
    {"AssetType": "Receipt", "RetentionClass": "KEEPER", "Locked": true},
    {"AssetType": "ApprovalSignature", "RetentionClass": "KEEPER", "Locked": true},
    {"AssetType": "VarianceReport", "RetentionClass": "REFERENCE", "Locked": false},
    {"AssetType": "PaymentConfirmation", "RetentionClass": "TRANSACTIONAL"}
  ]
}

Module-Specific Rules
3-Way Match (Mandatory Gate)
Before proceeding to payment:

Invoice amount matches PO (or variance approved).
Receipt amount matches invoice (proof of delivery/rendering per §56).
All three documented as assets.

Variance → Create VarianceReport, require budget authority approval, log as blocker until resolved.
Approval Chain (Warrant-Equivalent per §56)

Department head: Verify correctness/delivery.
Accounting: Confirm coding, budget availability.
Finance Director/Treasurer/Selectmen: Final authority per thresholds.

All approvals recorded as locked ApprovalSignature assets.
Payment Methods

Check, ACH, credit card, wire (per town policy).
PaymentConfirmation asset generated post-processing.

Budget Encumbrance
Pre-commit funds on PO creation; check unencumbered balance before approval/payment.

Asset Types Table

AssetTypeRetentionClassLocked?PurposeInvoiceKEEPERYesOriginal vendor-submitted invoicePurchaseOrderKEEPERYesPre-authorization to vendorReceiptKEEPERYesProof of goods/services received/renderedApprovalSignatureKEEPERYesDigitally signed approval (per §56)VarianceReportREFERENCENoDocumentation of differences & resolutionsPaymentConfirmationTRANSACTIONALNoBank/treasurer proof of disbursement

Decision Tables
Gate 1: 3-Way Match Variance

Invoice AmtPO AmtReceipt AmtMatch?Action / Blocker$1,000$1,000$1,000YES ✓Proceed$1,000$900$1,000NOVariance: Invoice exceeds PO; require approval$1,000$1,000$950NOVariance: Receipt under-delivered; investigate$1,100$1,000$1,100NOVariance: Over-spend; budget authority sign-off
Resolution: Approved variance → lock ApprovalSignature; unresolved → blocker.
Gate 2: Budget Availability
Requested AmtAvailableEncumberedUnencumberedProceed?Action$5,000$10,000$0$10,000YES ✓Proceed$5,000$3,000$0$3,000NOReject: Insufficient$5,000$8,000$5,000$3,000NOReject: Over-encumbered$5,000$10,000$6,000$4,000YES ✓Proceed (if sufficient)
Approval Matrix (Threshold-Based per §56 & Policy)

Amount RangeAuthority RequiredSignature Level$0 – $1,000Department HeadRequired$1,001 – $5,000Finance DirectorRequired$5,001 – $25,000TreasurerRequired$25,001+Board of SelectmenRequired

High-Level Process Flow
textInvoice Intake
  ↓
Locate / Match PO
  ↓
Locate / Match Receipt
  ↓
3-Way Match Gate?
  ├─ NO ──► Log Variance → Request Approval → Recheck (Blocker if unresolved)
  └─ YES ──► Budget Availability Gate?
               ├─ NO ──► Reject: Insufficient Funds
               └─ YES ──► Route Approval Chain (per Matrix)
                            ↓
                       Approval Signature(s) → LOCKED
                            ↓
                       Select Payment Method
                            ↓
                       Process Payment → Confirmation
                            ↓
                       GL Posting → Reconciliation
                            ↓
                       CLOSED (All Assets Locked, Audit Sealed)

Compliance Checklist (Before Closure)

 Invoice, PO, Receipt recorded and matched (or variance approved)
 3-way match gate passed (per §56 delivery verification)
 Budget availability/encumbrance confirmed
 Full approval chain completed (warrant-equivalent signatures)
 Payment method selected and processed
 PaymentConfirmation and GL posting verified
 Reconciliation complete
 Key assets (Invoice/PO/Receipt/Approvals) LOCKED
 All deadlines enforced (no unresolved blockers)
 Audit log complete and sealed
 Closure reason recorded (e.g., "Paid per M.G.L. c. 41 §56")


Module Expansion Notes
Future features:

Vendor master data sync/integration
Budget/ERP interface (e.g., via Polimorphic + SSI partnership for cloud accounting/AP automation)
Multi-currency handling
Recurring invoices (auto-sub-CASEs)
Credit/debit memo linkage
Audit/legal hold flags (prevent closure/destruction)
AI-assisted variance detection/routing (leverage Polimorphic workflows)


VAULT FISCAL fully inherits the CASE architecture (immutable ID, versioned scope, locked assets, enforced timers, sealed audit). Finance rules (3-way match, warrant-style approvals, budget gates) are enforced as module-specific invariants — delivering audit-proof, MA-compliant municipal finance operations.
