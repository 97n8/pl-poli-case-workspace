# VAULT CLERK Module — Charter

**Licensing and Clerk Work • Municipal Governance**

---

## Module Purpose

VAULT CLERK enforces licensing and clerk functions as CASE-based processes.

Examples:
* Contractor licenses
* Plumbing / electrical / gas permits
* Food service permits
* Marriage licenses
* Vital records
* Regulatory approvals

Each application is one CASE. Process is identical regardless of license type.

---

## Statutory Authority

**M.G.L. c. 101** — Licenses and Permits (varies by specific license)

Different license types have different timelines and review requirements. VAULT CLERK is modular: configure per license type.

---

## CASE Model for CLERK

### Identity

```json
{
  "CaseID": "clerk-2026-0145",
  "CaseType": "CLERK",
  "CreatedTimestamp": "2026-02-01T10:15:00Z",
  "CreatedBy": "clerk@town.example.com"
}
```

### Subject

```json
{
  "ApplicantIdentity": {
    "Name": "ABC Plumbing LLC",
    "Type": "Business"
  },
  "ApplicationType": "Contractor_License_Annual",
  "PropertyAddress": "123 Main Street" (if applicable)
}
```

### Scope

```json
{
  "ScopeDefinition": {
    "LicenseType": "Contractor",
    "ServiceArea": "Plumbing",
    "ApplicationPeriod": "2026-2027",
    "RequiredDocuments": ["Proof of Insurance", "Background Check", "Bonding"]
  },
  "ScopeVersion": 1
}
```

### Deadlines

```json
{
  "T10": {
    "DueDate": "2026-02-15",
    "Description": "Initial municipal review window"
  },
  "ApprovalDue": {
    "DueDate": "2026-03-01",
    "Description": "Inspection completion and approval decision"
  }
}
```

### Assets

```json
{
  "Assets": [
    {"AssetType": "Application", "RetentionClass": "REFERENCE"},
    {"AssetType": "ProofOfInsurance", "RetentionClass": "REFERENCE"},
    {"AssetType": "BackgroundCheckResult", "RetentionClass": "KEEPER"},
    {"AssetType": "InspectionReport", "RetentionClass": "KEEPER"},
    {"AssetType": "ApprovalCertificate", "RetentionClass": "KEEPER", "Locked": true}
  ]
}
```

---

## Module-Specific Rules

### License Types

Each license type may have different:
* Required documents
* Inspection requirements
* Approval authority
* Fee structure
* Renewal timeline

Configure these per license type. Core CASE structure remains identical.

### Approval Authority

Some licenses require:
* Health Inspector approval
* Building Inspector approval
* Board of Selectmen approval

CLERK records who approved and when.

### Denial

If application is denied:
* Reason must be documented
* Requester may request administrative review
* Appeal window may exist (per statute)

---

## Process Flow (High Level)

```
Intake → Review Requirements → Request Documents → 
  Inspection/Evaluation → Approval/Denial → Issuance → Closure
```

---

## Asset Types in CLERK

| Asset | Retention | Locked? | Purpose |
|-------|-----------|---------|---------|
| Application | REFERENCE | No | What was requested |
| SupportingDocuments | REFERENCE | No | Submitted by applicant |
| ApprovalCertificate | KEEPER | Yes | Issued license/approval |
| DenialLetter | KEEPER | Yes | If denied, reason |
| InspectionReport | KEEPER | Yes | Evidence of inspection |
| RenewalNotice | TRANSACTIONAL | No | Reminder to renew |

---

## Decision Tables

### Gate 1: Document Completeness

| Application Type | Required Documents | Missing → Action |
|------------------|-------------------|------------------|
| Contractor | Insurance, Background, Bonding | Request clarification; clock tolled |
| Plumbing | License, Insurance, Exam | Request clarification; clock tolled |
| Food Service | Health Cert, Insurance, Training | Request clarification; clock tolled |

---

### Gate 2: Inspection Needed?

| Application Type | Inspection Required? | If Required, By When? |
|------------------|-----|---|
| Contractor | Yes | Within 14 days |
| Plumbing | Yes | Within 10 days |
| Food Service | Yes | Within 7 days |
| Mail-in License | No | N/A |

---

### Gate 3: Approval Authority

| Application Type | Authority | Signature Required? |
|-----|----|----|
| Contractor | Building Inspector | Yes (locked) |
| Plumbing | Plumbing Inspector | Yes (locked) |
| Food Service | Health Director | Yes (locked) |

---

## Process Flow (High-Level)

```
[Intake]
  ↓
[Completeness Check] ← Clarification loop if needed
  ↓
[Inspection Needed?]
  ├─ YES → [Schedule Inspection] → [Conduct Inspection] → [Inspection Results]
  └─ NO → [Skip to Approval Gate]
  ↓
[Approval Authority Reviews] → [Approve / Deny Decision]
  ├─ Approve → [Issue Certificate] → [LOCKED]
  ├─ Deny → [Issue Denial Letter] → [LOCKED]
  └─ Request Info → [Clarification Loop]
  ↓
[CLOSED]
```

---

## Compliance Checklist for CLERK CASE

Before closing:

- [ ] Application received and recorded
- [ ] All required documents collected (completeness check passed)
- [ ] Inspection scheduled (if required)
- [ ] Inspection completed and results recorded
- [ ] Approval authority reviewed application
- [ ] Approval or denial decision made and documented
- [ ] Certificate issued OR denial letter sent (LOCKED)
- [ ] Key assets locked (approval/denial, inspection report)
- [ ] Audit log sealed
- [ ] No transition blockers remain

---

## Module Expansion Notes

Detailed implementation will include:
* Complete license-type matrix
* Inspection procedures and forms
* Appeals process (if applicable per statute)
* Renewal workflows (v1.1 scope)
* Multi-step approvals (board sign-off, etc.)

---

*VAULT CLERK follows the same CASE architecture as PRR. Only license-type-specific rules vary.*
