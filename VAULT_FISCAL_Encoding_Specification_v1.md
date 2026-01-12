ext# VAULT FISCAL™ — Encoding Specification
**Implementation Contract • Version 1.0**
**Authority:** PublicLogic™
**Runtime Partner:** Polimorphic (with SSI integration for finance backend)
**Date:** January 12, 2026

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
State Transition Matrix

FromToConditionAction / EnforcementPolimorphic Mapping SuggestionINTAKEVALIDATIONInvoice asset receivedValidate required fields (vendor, amount, invoice#, etc.)Auto-trigger on document uploadVALIDATIONMATCHINGAll fields validLocate/match to PO & Receipt assetsWorkflow step: document linkingMATCHINGVARIANCE_REVIEWAmounts mismatch (±0.01 tolerance)Flag variance; create VarianceReport asset; set blockerAI-assisted variance detectionMATCHINGBUDGET_CHECKAmounts matchProceed to budget encumbrance checkSSI integration hook for encumbrance queryVARIANCE_REVIEWBUDGET_CHECKVariance approved/explainedLock ApprovalSignature (budget authority); clear blockerMulti-level approval routingVARIANCE_REVIEWINTAKEUnresolved / rejectedRequest resubmission; notify vendorAuto-email/escalationBUDGET_CHECKAPPROVAL_ROUTINGUnencumbered budget ≥ amountRoute per Approval MatrixConditional routing + notificationsBUDGET_CHECKINTAKEInsufficient budgetEscalate (e.g., to Finance Director); block transitionAI escalation pathAPPROVAL_ROUTINGAPPROVAL_PENDINGAuthority assignedWait for digital signatureTask assignment in CRM/workflowsAPPROVAL_PENDINGAPPROVEDAuthority approvesLock ApprovalSignature asset; clear blockersLock on approval eventAPPROVAL_PENDINGINTAKEAuthority deniesRequest resubmission; log denial reasonRejection loop with notesAPPROVEDPAYMENT_PROCESSINGPayment method selectedGenerate ACH/check; record PaymentConfirmationSSI payment initiationPAYMENT_PROCESSINGPAYMENT_COMPLETEPayment sent/confirmedUpdate status; notify treasurerBank integration triggerPAYMENT_COMPLETERECONCILIATIONBank confirmation receivedReconcile to GL via SSIAuto-reconciliation hookRECONCILIATIONCLOSEDReconciliation clearedLock GL posting; seal audit log; set ClosureReason ("Paid per M.G.L. c. 41 §56")Final lock + archive
Rule: Transitions forward only; blockers must be empty to proceed. All state changes logged in AuditLog (immutable append).

Part II: JSON Schemas
Core Invoice CASE Schema (extends base CASE)
JSON{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "VAULT FISCAL Invoice CASE",
  "type": "object",
  "required": ["caseId", "vendor", "invoice", "matching", "approval"],
  "properties": {
    "caseId": { "type": "string", "pattern": "^fiscal-[0-9]{4}-[0-9]{4}$" },
    "vendor": {
      "type": "object",
      "properties": {
        "vendorId": { "type": "string" },
        "status": { "type": "string", "enum": ["ACTIVE", "INACTIVE", "DEBARRED"] }
      },
      "required": ["vendorId", "status"]
    },
    "invoice": {
      "type": "object",
      "properties": {
        "invoiceNumber": { "type": "string" },
        "amount": { "type": "number", "minimum": 0.01 },
        "date": { "type": "string", "format": "date-time" }
      },
      "required": ["invoiceNumber", "amount"]
    },
    "matching": {
      "type": "object",
      "properties": {
        "poAmount": { "type": "number" },
        "receiptAmount": { "type": "number" },
        "matchStatus": { "type": "string", "enum": ["MATCHED", "VARIANCE", "UNMATCHED"] },
        "toleranceUsed": { "type": "boolean" }
      },
      "required": ["matchStatus"]
    },
    "approval": {
      "type": "object",
      "properties": {
        "approvalStatus": { "type": "string", "enum": ["PENDING", "APPROVED", "DENIED"] },
        "approvalSignatureLocked": { "type": "boolean" },
        "authorityLevel": { "type": "string" }
      },
      "required": ["approvalStatus", "approvalSignatureLocked"]
    }
  }
}

Part III: Critical Validation Rules (Pseudocode)
3-Way Match Verification
texttolerance = 0.01
IF ABS(Invoice.amount - POAmount) <= tolerance AND ABS(Invoice.amount - ReceiptAmount) <= tolerance:
  matching.matchStatus = "MATCHED"
  matching.toleranceUsed = (ABS diff > 0)
  Proceed to BUDGET_CHECK
ELSE:
  matching.matchStatus = "VARIANCE" or "UNMATCHED"
  Transition to VARIANCE_REVIEW
  Set blocker: "E_3WAY_MATCH_FAILED"
Budget Availability (via SSI integration)
textIF currentUnencumberedBudget(GLCode) >= Invoice.amount:
  Proceed to APPROVAL_ROUTING
ELSE:
  Set blocker: "E_BUDGET_INSUFFICIENT"
  Transition to INTAKE or escalate
Approval Authority Routing
textIF Invoice.amount <= 1000:
  authority = "Department Head"
ELSIF Invoice.amount <= 5000:
  authority = "Finance Director"
ELSIF Invoice.amount <= 25000:
  authority = "Treasurer"
ELSE:
  authority = "Board of Selectmen"
approval.authorityLevel = authority
Transition to APPROVAL_ROUTING
Locked Assets
textIF approval.approvalSignatureLocked == true OR GLPosted == true:
  ON edit attempt: Return E_CANNOT_EDIT_LOCKED
  Suggest: Create new version / reversing entry (audit preserved)

Part IV: Timer Algorithms (Enforced)
T_PAYMENT_DUE
textdue = Invoice.date + VendorPaymentTerms (e.g., Net 30 calendar days)
IF currentDate > due AND status != "PAYMENT_COMPLETE":
  Log event: "Payment overdue"
  Notify: Treasurer / Finance Director
  Flag: Potential interest (per contract/policy)
T_APPROVAL_WINDOW
textwindow = Invoice.receivedDate + 5 business days
IF currentDate > window AND approval.approvalStatus == "PENDING":
  Escalate: Next level (e.g., Finance Director → Treasurer)
  Log: "Approval window exceeded"
  Notify: Current + escalated authority
T_RECONCILIATION
textreconDue = PaymentClearedDate + 30 calendar days
IF currentDate > reconDue AND status != "RECONCILIATION" or "CLOSED":
  Log: "Reconciliation overdue"
  Flag: Variance investigation required
  Notify: Accounting

Part V: Error Handling Table

CodeConditionAction / ResponseE_3WAY_MATCH_FAILEDAmounts mismatchBlock payment; request variance explanationE_BUDGET_INSUFFICIENTUnencumbered < amountBlock; escalate for budget amendment/reviewE_VENDOR_INACTIVEVendor status != ACTIVEBlock; escalate to procurementE_DUPLICATE_DETECTEDMatching invoice already paidBlock; investigate duplicate payment riskE_APPROVAL_DENIEDAuthority rejectsTransition back; request resubmissionE_CANNOT_EDIT_LOCKEDEdit on locked assetReturn error; explain immutability; suggest reversal entry

Part VI: Deployment Checklist

 3-way match scenarios tested (match, variance ±0.01, mismatch)
 Budget encumbrance / availability via SSI tested
 Approval routing & escalation tested (all matrix thresholds)
 Payment methods integration tested (ACH/check via SSI)
 GL posting & reconciliation tested (SSI sync)
 Timer enforcement & notifications tested
 Segregation of duties (e.g., approver ≠ processor)
 Vendor debarment / duplicate checks integrated
 All error codes surfaced & handled
 Locked assets immutable; reversals create new entries
 Audit log sealed on CLOSED; full traceability
 Polimorphic workflow mapping validated (routing, AI notifications, status sync)


Build exactly as specified. VAULT FISCAL v1.0 delivers warrant-compliant (§56), audit-proof finance operations with Polimorphic front-end automation and SSI backend integration.
