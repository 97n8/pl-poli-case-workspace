# VAULT CLERK Module — Charter (Final v1.0)

**Licensing, Permits & Clerk Functions • Municipal Governance**

Current Date Reference: January 2026

---

## Module Purpose

VAULT CLERK manages all licensing, permitting, and clerk-issued record processes as standardized **CASE** objects.

Examples of supported processes:
- Marriage intentions & licenses
- Vital records requests & certified copies
- Business certificates (DBA/fictitious names)
- Contractor / home improvement contractor registrations
- Plumbing, gas fitting, electrical, and other specialty trade permits
- Food service / retail food establishment permits
- Dog licenses
- Raffle / bazaar permits
- Alcohol license applications (intake only)
- Transient vendor / hawker / peddler licenses

Each application or request = **one CASE**. Core process structure remains identical across types; differences are handled via configuration.

---

## Statutory & Regulatory Authority (Primary Examples)

| Process Type                  | Primary Statute(s)                          | Local Oversight                     | Typical Issuing/Approving Authority    |
|-------------------------------|---------------------------------------------|-------------------------------------|----------------------------------------|
| Marriage License              | M.G.L. c. 207                               | Town/City Clerk                     | Clerk / Assistant Clerk                |
| Vital Records (copies)        | M.G.L. c. 46; 950 CMR 100                   | Town/City Clerk (local registrar)   | Clerk / RVRS interface                 |
| Business Certificate (DBA)    | M.G.L. c. 110 § 5                           | Town/City Clerk                     | Clerk                                  |
| Dog License                   | M.G.L. c. 140 §§ 136–174                    | Town/City Clerk                     | Clerk                                  |
| Plumbing / Gas / Electrical   | M.G.L. c. 142, c. 143 §§ 3, 3Z; 248 CMR     | Local plumbing/wire/gas inspectors  | Inspector(s) + Clerk issuance          |
| Food Service Permit           | M.G.L. c. 94; 105 CMR 590 (state sanitary code) | Board of Health                     | Health Director / Agent                |
| Contractor Registration       | M.G.L. c. 142A; local bylaws; BBRS          | Building Department / Clerk         | Building Inspector / Clerk             |
| Transient Vendor / Hawker     | M.G.L. c. 101                               | Selectmen / Clerk                   | Selectmen / Clerk                      |

VAULT CLERK is **license-type configurable** to reflect statutory timelines, required documents, fees, inspection needs, and approval authorities.

---

## CASE Model for CLERK

### Identity
```json
{
  "CaseID": "clerk-2026-0145",
  "CaseType": "CLERK",
  "SubType": "Marriage_License",              // or "Food_Permit", "Plumbing_Permit", etc.
  "CreatedTimestamp": "2026-02-01T10:15:00Z",
  "CreatedBy": "clerk@town.example.gov"
}
```

### Subject
```json
{
  "Applicant": {
    "Name": "Jane Doe & John Smith",
    "Type": "Individual",                     // or "Business", "Organization"
    "Contact": { "Email": "...", "Phone": "..." }
  },
  "ApplicationType": "Marriage_Intentions",
  "RelatedProperty": "N/A",                   // or full address if applicable
  "PriorCaseID": "clerk-2025-0892"            // for renewals / related matters
}
```

### Scope
```json
{
  "ScopeDefinition": {
    "LicenseType": "Marriage",
    "StatutoryAuthority": "M.G.L. c. 207 §§ 19–37",
    "LocalBylaw": null,
    "ApplicationPeriod": "2026",
    "RequiredDocuments": ["PhotoID", "BirthCertificate", "DivorceDecree_if_applicable"],
    "Fee": 50.00,
    "FeePaid": false,
    "InspectionRequired": false,
    "ApprovalAuthorities": ["Clerk"]
  },
  "ScopeVersion": 1
}
```

### Deadlines & Milestones
```json
{
  "Milestones": {
    "T10_Completeness": { "DueDate": "2026-02-08", "Description": "Initial review window" },
    "InspectionDue":      null,
    "DecisionDue":        { "DueDate": "2026-02-15", "Description": "Final issuance/denial" },
    "AppealWindow":       { "Days": 30, "Statute": "M.G.L. c. 30A if applicable" }
  }
}
```

### Assets
```json
{
  "Assets": [
    { "AssetType": "ApplicationForm",      "RetentionClass": "REFERENCE" },
    { "AssetType": "ProofOfAge_ID",        "RetentionClass": "REFERENCE" },
    { "AssetType": "InspectionReport",     "RetentionClass": "KEEPER", "Locked": true },
    { "AssetType": "MarriageCertificate",  "RetentionClass": "KEEPER", "Locked": true },
    { "AssetType": "DenialLetter",         "RetentionClass": "KEEPER", "Locked": true },
    { "AssetType": "RenewalNotice",        "RetentionClass": "TRANSACTIONAL" }
  ]
}
```

---

## Module-Specific Rules

- **Configurable per LicenseType**:
  - Required documents list
  - Fee amount & payment status tracking
  - Inspection requirement + inspector role
  - Approval authority chain (single or multi-step)
  - Statutory / bylaw deadlines
  - Renewal cycle & auto-notice generation

- **Approval Authority Chain** (example for multi-step):
  ```json
  "Approvals": [
    { "Authority": "Health Agent",   "Status": "Pending", "DueDate": "2026-02-10" },
    { "Authority": "Clerk",          "Status": "Pending" }
  ]
  ```

- **Denial & Appeal**:
  - Denial requires documented reason(s)
  - DenialLetter asset must be generated & locked
  - Appeal window tracked (if statutory)
  - Appeal may trigger new sub-case or reopen

---

## High-Level Process Flow

```
Intake (application + fee)
  ↓
Completeness Check → [Clarification Request Loop] (clock tolled)
  ↓
Inspection Required?
  ├─ Yes → Schedule → Conduct → Record Results
  └─ No  ────────────────────────────────┐
                                         ↓
Approval Authority(ies) Review
  ↓
Decision: Approve / Deny / More Info
  ├─ Approve → Issue Certificate → Lock assets → Notify
  ├─ Deny    → Issue Denial Letter → Lock → Notify + Appeal rights
  └─ More Info → Clarification Loop
  ↓
Closure Checklist → Seal audit log → CLOSED
```

---

## Asset Retention & Locking Summary

| Asset Type            | Retention Class | Locked on Closure? | Purpose                              |
|-----------------------|-----------------|---------------------|--------------------------------------|
| Application           | REFERENCE       | No                  | Original request                     |
| Supporting Docs       | REFERENCE       | No                  | Applicant submissions                |
| Inspection Report     | KEEPER          | Yes                 | Evidence of compliance               |
| Approval Certificate  | KEEPER          | Yes                 | Official license/permit/record       |
| Denial Letter         | KEEPER          | Yes                 | Documented rationale + appeal rights |
| Renewal Notice        | TRANSACTIONAL   | No                  | Automated reminders                  |

---

## Gate Decision Tables (Examples – Configurable)

**Gate 1: Completeness**

| Type              | Required Documents Example                  | Missing Action                  |
|-------------------|---------------------------------------------|---------------------------------|
| Marriage          | IDs, birth certs, prior marriage proof      | Request + toll clock            |
| Food Service      | Plans, ServSafe, insurance                  | Request + toll clock            |
| Plumbing Permit   | Licensed plumber name, insurance            | Request + toll clock            |

**Gate 2: Inspection**

| Type              | Inspection? | Typical Inspector     | Target Window |
|-------------------|-------------|-----------------------|---------------|
| Food Service      | Yes         | Health Agent          | 7–10 days     |
| Plumbing Permit   | Yes         | Plumbing Inspector    | 10 days       |
| Marriage          | No          | N/A                   | N/A           |

**Gate 3: Final Approval**

| Type              | Primary Authority       | Signature / e-Sign Required? |
|-------------------|-------------------------|------------------------------|
| Marriage          | Clerk                   | Yes                          |
| Food Service      | Health Director         | Yes                          |
| Plumbing Permit   | Plumbing Inspector      | Yes                          |
| Transient Vendor  | Selectmen (or Clerk)    | Yes (multi if Selectmen)     |

---

## Closure Compliance Checklist

Before case can be CLOSED:

- [ ] Application received, timestamped, fee recorded
- [ ] Completeness check passed (all required documents)
- [ ] Inspection completed & results recorded (if required)
- [ ] All required approvals obtained & logged
- [ ] Decision (approve/deny) documented
- [ ] Certificate issued OR denial letter sent
- [ ] All KEEPER assets locked
- [ ] Applicant notified
- [ ] Audit trail sealed
- [ ] No open blockers

---

## Planned Expansion (v1.1+)

- Full license-type configuration matrix
- Electronic signature integration (M.G.L. c. 110G)
- Renewal auto-workflow with prior-case linking
- Multi-board sequential approvals
- Vital records privacy restrictions & RVRS export
- Appeal / hearing sub-case workflow
- Fee receipt & refund tracking
- Public hearing flag & notice generation

---

VAULT CLERK uses the shared CASE architecture with PRR and other modules.  
Only license-type-specific configuration tables vary.

This charter reflects Massachusetts General Laws, state regulations, and common municipal clerk / department practices as of January 2026.
