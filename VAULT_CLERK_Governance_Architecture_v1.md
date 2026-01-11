# VAULT CLERK™ — Governance Architecture & Domain Model

**Framework-as-a-Standard™ • Version 1.0**  
**Authority:** PublicLogic™  
**Governing Law:** M.G.L. c. 101 (Licenses and Permits, varies by license type)

---

## Document Classification

| Attribute | Value |
|-----------|-------|
| **Type** | Governance Architecture |
| **Status** | Authoritative |
| **Confidentiality** | Internal / NDA Partners |
| **Owner** | PublicLogic™ |
| **Encoding Spec** | VAULT_CLERK_Encoding_Specification_v1.md |

---

## Purpose

VAULT CLERK™ defines the **governed, enforcement-first operating system** for Massachusetts municipal licensing and permitting.

This document converts licensing statutes and municipal ordinances into **structural requirements** so that:

1. **Applications are processed in defined timelines** — deadlines enforce, not remind
2. **Approvals and denials are documented and locked** — decisions are defensible
3. **Inspections are recorded and verified** — compliance is auditable
4. **Records are immutable where required** — decisions cannot be altered
5. **Risk is carried by structure, not individuals** — operators execute within protected boundaries

**Core Principle:** If a license decision isn't recordable, transferable, and defensible, it isn't safe.

---

## Document Hierarchy

VAULT CLERK™ operates through three document tiers:

| Tier | Document | Purpose | Audience |
|------|----------|---------|----------|
| **1** | **Governance Architecture** (this document) | Statutory authority, enforcement principles, domain model | PublicLogic, executives, legal |
| **2** | **Encoding Specification** | Implementation contract — schemas, state machine, algorithms | Runtime partners (Polimorphic) |
| **3** | **Derivative Artifacts** | SOPs, training, UI specs, local policy | Municipal staff, end users |

**Compliance Rule:** Tier 2 implements Tier 1. Tier 3 layers on Tier 2. Conflicts resolve upward.

---

## Part I: Statutory Authority

### Governing Law

VAULT CLERK™ enforces licensing requirements across multiple statutes, depending on license type:

| License Type | Primary Statute | Secondary Reqs | Enforcement |
|---|---|---|---|
| **Contractor** | M.G.L. c. 101 | Insurance, bonding, background | VAULT CLERK manages CASE; inspection required |
| **Plumbing** | M.G.L. c. 142, §19 | License exam, insurance | VAULT CLERK manages CASE; inspection required |
| **Electrical** | M.G.L. c. 141 | License exam, insurance | VAULT CLERK manages CASE; inspection required |
| **Food Service** | M.G.L. c. 94, §305 | Health cert, training | VAULT CLERK manages CASE; health dept inspection |
| **Building Permit** | M.G.L. c. 143 | Building code review | VAULT CLERK manages CASE; building inspector approval |

### Common Statutory Requirements Across License Types

| Requirement | Statute | VAULT CLERK Enforcement |
|---|---|---|
| Application must be in writing | M.G.L. c. 101 | IntakeRecord asset (REFERENCE) |
| Decision deadline (varies: 10-30 days) | M.G.L. c. 101, § 4 (varies by type) | T_DECISION timer; gate enforced |
| Decision must be in writing | M.G.L. c. 101, § 5 | ApprovalLetter or DenialLetter (LOCKED, KEEPER) |
| Written explanation for denial | M.G.L. c. 101 | DenialLetter must cite specific reason |
| Appeal right (30 days) | M.G.L. c. 101, § 8 | T90 timer (post-closure); logged |
| License expiration and renewal | M.G.L. c. 101 (varies) | RenewalDate set at issuance; renewal tracked separately (v1.1) |

---

## Part II: Enforcement Architecture

### CASE Lifecycle for Licensing

A CLERK CASE follows this progression:

```
INTAKE
  ↓ [Application received; completeness checked]
ASSESSMENT
  ↓ [Inspection needed? Documents complete? Authority determined?]
INSPECTION (if required)
  ↓ [Inspector conducts; results recorded]
REVIEW
  ↓ [Approval authority examines documents + inspection results]
APPROVAL/DENIAL
  ↓ [Decision made; letter drafted]
ISSUANCE (if approved) or CLOSURE (if denied)
  ↓ [License issued or denial finalized; LOCKED]
CLOSED
```

### License-Specific Decision Trees

#### Contractor License

**T_INTAKE to T_ASSESSMENT:** 2 business days
- Application completeness check: Insurance, bonding, background check forms received?
- If incomplete: Request clarification (tolling event)

**T_ASSESSMENT to T_INSPECTION:** 5 business days
- Building inspector must schedule and conduct site inspection
- Inspection report recorded (KEEPER asset)

**T_DECISION:** 20 business days from intake
- If all documents + inspection passed: Approve
- If any failure: Deny with written reason
- Consequence if missed: License application deemed approved (statutory default)

---

#### Plumbing License

**T_INTAKE:** 3 business days
- Completeness: License exam result, proof of apprenticeship, insurance

**T_EXAM_VERIFY:** 5 business days
- Verify exam was passed; contact exam board if needed

**T_INSPECTION:** 10 business days
- Plumbing inspector observes qualifications; conducts practical test (if required by local rule)

**T_DECISION:** 15 business days from intake
- Decision is automatic if exam + inspection pass
- Denial only if exam failed or inspection identified deficiency

---

#### Food Service License

**T_INTAKE:** 2 business days
- Completeness: Health certificate, food safety training, insurance

**T_HEALTH_DEPT:** 5 business days
- Health department conducts walk-through inspection
- Violations recorded (if any)

**T_CORRECTION:** 5 business days
- Applicant corrects violations (if any)
- Health dept re-inspects

**T_DECISION:** 20 business days from intake
- Approve if health dept clearance obtained
- Deny if critical violations cannot be corrected

---

### Timer Model

All CLERK applications have:

| Timer | Definition | Consequence |
|---|---|---|
| **T_INTAKE** | Time to verify application completeness | If missed: Request clarification (tolled) |
| **T_DECISION** | Time to issue approval or denial | If missed: License is **deemed approved** (statutory default) |
| **T_INSPECTION** | Time to complete required inspection | If missed: Request extension from applicant |
| **T90** | Appeal window (post-closure, 30 calendar days typically) | If expired: Appeal right is lost |

**Key difference from PRR:** License statutes often have "deemed approved" provisions. If decision is not made by deadline, license is automatically approved. This is enforced by gate.

---

### Automatic Enforcement Rules

| Rule | Trigger | Action |
|---|---|---|
| **Deemed Approval** | T_DECISION deadline missed AND ApprovalStatus not set | Set ApprovalStatus = APPROVED; log; issue license |
| **Inspection Gate** | T_INSPECTION deadline exceeded | Block approval until inspection complete or extension granted |
| **Denial Lock** | Denial letter generated | Mark DenialLetter as LOCKED; cannot edit reason |
| **Appeal Window Close** | 30 calendar days after closure | Close appeal window; log; no appeals accepted after |

---

### Inspection Verification Rules

Inspections are critical gates. Rules:

1. **Inspection is mandatory** (for specified license types)
2. **Inspection must be documented** (InspectionReport asset, KEEPER)
3. **Inspection result (pass/fail/conditional)** determines approval/denial
4. **Conditional approval** (pass with corrections) creates secondary CASE for follow-up inspection

---

## Part III: Domain Model

### Entity Relationship Overview

```
CASE (root container)
├── ApplicationType (enum: Contractor, Plumbing, Electrical, Food, etc.)
├── Applicant (business or individual)
├── ApplicationStatus (intake, assessment, inspection, review, approval/denial, closed)
├── InspectionRequired (boolean)
├── InspectionSchedule (date, inspector name)
├── Deadlines (T_INTAKE, T_DECISION, T_INSPECTION, T90)
├── Assets (Application, Insurance, BackgroundCheck, InspectionReport, ApprovalLetter/DenialLetter)
├── ApprovalAuthority (Building Inspector, Health Director, Licensing Board, etc.)
├── LicenseExpiry (if approved)
└── AuditLog (all decisions, inspections, approvals)
```

### Entity Definitions

#### CASE

The root container for a single licensing application.

| Attribute | Type | Description |
|---|---|---|
| CaseID | UUID | Immutable unique identifier |
| ApplicationType | Enum | Contractor, Plumbing, Electrical, Food, Building, etc. |
| Applicant | Object | Name, address, contact, entity type (business/individual) |
| ApplicationDate | Date | When application was submitted |
| EffectiveReceiptDate | Date | Computed from ApplicationDate (shifts if weekend/holiday) |
| InspectionRequired | Boolean | Does this license type require inspection? |
| ApprovalAuthority | String | Who approves (Building Inspector, Health Director, etc.) |
| Deadlines | Object | T_INTAKE, T_DECISION, T_INSPECTION, T90 |
| LicenseType | String | Contractor, Plumbing, etc. (mirrors ApplicationType) |
| LicenseExpiry | Date (nullable) | When license expires (if approved) |
| ApprovalStatus | Enum | PENDING, APPROVED, DENIED, DEEMED_APPROVED |
| ClosureReason | Enum | Approved, Denied, Deemed_Approved, Withdrawn (v1.1) |
| Assets | Array | All documents (application, inspection report, approval letter, etc.) |
| AuditLog | Array | Immutable activity record |

#### ApplicationType Configuration

Each license type has:

```json
{
  "ApplicationType": "Contractor",
  "StatutoryAuthority": "M.G.L. c. 101",
  "RequiredDocuments": ["Insurance", "Bonding", "BackgroundCheck"],
  "InspectionRequired": true,
  "InspectionAuthority": "Building Inspector",
  "DecisionDeadline": 20,  // business days
  "AppealDeadline": 30,  // calendar days
  "DeemedApprovedDefault": true,
  "FeeRequired": true,
  "FeeAmount": 150.00,
  "RenewalRequired": true,
  "RenewalInterval": 3  // years
}
```

#### Assets in CLERK CASEs

| Asset Type | Retention | Locked? | Purpose |
|---|---|---|---|
| Application | REFERENCE | No | Original application form |
| SupportingDocuments | REFERENCE | No | Insurance, bonding, background check, etc. |
| ApprovalLetter | KEEPER | Yes | Issued license approval (statutory record) |
| DenialLetter | KEEPER | Yes | Denial with reason (statutory record) |
| InspectionReport | KEEPER | Yes | Inspector's findings and recommendations |
| LicenseCertificate | KEEPER | Yes | Issued license (if approved) |
| RenewalNotice | TRANSACTIONAL | No | Reminder to renew |

---

## Part IV: Critical Gates

### Gate 1: Completeness Check

**When:** T_INTAKE deadline
**Condition:** All required documents received for this license type
**Pass:** Proceed to assessment
**Fail:** Request clarification; clock tolled

---

### Gate 2: Inspection Decision

**When:** T_ASSESSMENT
**Condition:** Does this license type require inspection?
**YES:** Schedule inspection; set T_INSPECTION deadline
**NO:** Skip to approval/denial assessment

---

### Gate 3: Inspection Results

**When:** T_INSPECTION deadline
**Condition:** Inspection completed; results documented
**PASS:** Proceed to approval gate
**FAIL/CONDITIONAL:** Request corrections from applicant (tolled)

---

### Gate 4: Approval Authority Review

**When:** T_DECISION deadline
**Condition:** All documents, inspections, background checks completed
**APPROVE:** Generate ApprovalLetter (LOCKED); issue license
**DENY:** Generate DenialLetter (LOCKED); cite reason
**MISSED:** Deemed Approved (statutory default); auto-approve

---

### Gate 5: Appeal Window

**When:** Post-closure (T90)
**Condition:** 30 calendar days from closure
**APPEAL ATTEMPTED WITHIN T90:** Accept appeal; escalate (v1.1)
**APPEAL ATTEMPTED AFTER T90:** Reject; window expired

---

## Part V: Implementation Boundaries

### PublicLogic™ Responsibility

- Framework definition and governance authority
- Statutory interpretation and enforcement rules
- Domain model and entity definitions
- Process architecture and state definitions
- Compliance validation and audit requirements

### Runtime Partner Responsibility (Polimorphic)

- State machine implementation per Encoding Specification
- Data schema implementation per JSON schemas
- Timer computation per algorithms
- Inspection scheduling and verification
- Integration with inspection authority systems
- User interface (layered on top)

### Municipal Authority Responsibility

- License type configuration (which documents, fees, inspectors)
- Inspector designation and contact
- Fees structure (if applicable)
- Local extensions to statute (if applicable)

### Governance Gap Protocol

If encoding surfaces ambiguity, introduces dependency, or creates new risk:

1. **Halt** — deployment stops immediately
2. **Escalate** — partner notifies PublicLogic
3. **Resolve** — PublicLogic issues clarification or amends spec
4. **Resume** — encoding continues with clarification

**Stop Rule:** If encoding introduces dependency or new risk, deployment halts.

---

## Part VI: Compliance Requirements

### Audit Trail Requirements

Every CLERK CASE must maintain an immutable audit log capturing:

- All state transitions with actor and timestamp
- All asset creation and locking events
- All inspection scheduling and completion events
- All approval/denial decisions
- All enforcement rule executions
- All error conditions

### Retention Requirements

| Record Type | Minimum Retention |
|---|---|
| CASE container | 7 years from closure (or per statute) |
| ApprovalLetter / DenialLetter | Permanent (KEEPER) |
| InspectionReport | Permanent (KEEPER) |
| SupportingDocuments | 7 years (REFERENCE) |
| Audit log | 7 years from closure |

**Note:** Some license types have longer retention (e.g., Building Permits may require 10+ years). Configure per statute.

---

## Part VII: Change Control

### Amendment Authority

| Change Type | Authority | Process |
|---|---|---|
| Statutory interpretation | PublicLogic | Legal review, version increment |
| Domain model change | PublicLogic | Architecture review, encoding spec update |
| Enforcement rule change | PublicLogic | Compliance review, implementation validation |
| License type configuration | Municipal + PublicLogic | Configuration review, no spec change needed |

### Version History

| Version | Date | Author | Changes |
|---|---|---|---|
| 1.0 | 2026-01-15 | PublicLogic | Initial release |

### Review Schedule

- **Quarterly:** Compliance validation against active cases
- **Annually:** Full architecture review
- **On demand:** Material law change, enforcement failure, audit finding

---

## Appendix A: Massachusetts License Types (Exemplar)

Common licenses managed by municipalities:

| Type | Statute | Authority | Inspection | Renewal | Notes |
|---|---|---|---|---|---|
| Contractor | M.G.L. c. 101 | Building Inspector | Yes | 3 years | Varies by trade |
| Plumbing | M.G.L. c. 142 | Plumbing Inspector | Yes | 3 years | Exam required |
| Electrical | M.G.L. c. 141 | Electrical Inspector | Yes | 3 years | Exam required |
| Food Service | M.G.L. c. 94 | Health Dept | Yes | 1 year | Annual reinspection |
| Building Permit | M.G.L. c. 143 | Building Inspector | Yes | N/A | Per project |

---

## Appendix B: Glossary (CLERK-Specific Terms)

| Term | Definition |
|---|---|
| **ApprovalAuthority** | Municipal official authorized to approve license (Building Inspector, Health Director, etc.) |
| **Deemed Approved** | Automatic approval when decision deadline is missed (statutory default) |
| **Inspection** | On-site or practical examination required for certain license types |
| **License Expiry** | Date when issued license is no longer valid; renewal may be required |
| **T_DECISION** | Deadline by which approval or denial decision must be made |
| **T_INSPECTION** | Deadline by which inspection must be scheduled and completed |
| **T_INTAKE** | Deadline by which completeness of application must be verified |

---

*This document is the authoritative governance architecture for VAULT CLERK™. Implementation must conform to the companion Encoding Specification. Deviations require written approval from PublicLogic.*
