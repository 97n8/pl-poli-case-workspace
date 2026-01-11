# VAULT FIX™ — Governance Architecture & Domain Model

**Framework-as-a-Standard™ • Version 1.0**  
**Authority:** PublicLogic™  
**Governing Law:** M.G.L. c. 30, §39K (Capital Asset Management)

---

## Document Classification

| Attribute | Value |
|-----------|-------|
| **Type** | Governance Architecture |
| **Status** | Authoritative |
| **Confidentiality** | Internal / NDA Partners |
| **Owner** | PublicLogic™ |

---

## Purpose

VAULT FIX™ defines the **governed, enforcement-first operating system** for municipal work orders, maintenance requests, and capital project tracking.

**Core Principle:** Work management must be auditable, traceable, and verifiable so that facility maintenance is defensible and capital assets are properly stewarded.

---

## Part I: Statutory Authority

### Governing Law

| Statute | Authority | VAULT FIX Enforcement |
|---|---|---|
| **M.G.L. c. 30, §39K** | Capital Asset Management | Asset tracking, work completion verification |
| **M.G.L. c. 149** | Labor law compliance | Contractor licensing, wage compliance |
| **Local Facilities Policy** | Maintenance standards | Work approval, completion verification |

---

## Part II: Enforcement Architecture

### CASE Lifecycle for Work Orders

```
INTAKE
  ↓ [Request received; priority assigned]
TRIAGE
  ↓ [Assess scope; determine if emergency/routine]
SCHEDULING
  ↓ [Schedule work; assign technician or contractor]
IN_PROGRESS
  ↓ [Work is being performed]
COMPLETION_REVIEW
  ↓ [Verify work completed per spec]
CLOSURE
  ↓ [Document completion; close CASE; lock records]
```

### Work Priority Classification

| Priority | SLA Response | SLA Completion | Examples |
|---|---|---|---|
| **CRITICAL** | 2 hours | 24 hours | Safety hazard, building system down |
| **HIGH** | 4 hours | 5 business days | Significant disruption; workaround available |
| **NORMAL** | 24 hours | 14 business days | Routine maintenance, minor issue |
| **LOW** | 5 business days | 30 business days | Cosmetic, improvement, non-urgent |

### Assignment Rules

Work is assigned based on:
1. **Skill required** (electrical, plumbing, carpentry, HVAC, IT, etc.)
2. **In-house availability** (staff hours, expertise)
3. **Contractor availability** (if in-house unavailable)
4. **Priority classification** (CRITICAL gets immediate attention)

### Completion Verification

Each work order requires verification **before closure**:

| Work Type | Verified By | Method |
|---|---|---|
| Electrical | Licensed electrician or building inspector | Inspection + sign-off |
| Plumbing | Licensed plumber or building inspector | Inspection + sign-off |
| HVAC | Facilities manager + requester | Testing + confirmation |
| Carpentry | Facilities manager + requester | Visual + requester sign-off |
| IT | IT lead + requester | Testing + requester sign-off |

**Rule:** Work cannot close until verification is documented and locked.

---

## Part III: Domain Model

### Entity Relationship Overview

```
CASE (root container)
├── RequestType (Work order, maintenance request, capital project, etc.)
├── Requester (who submitted request)
├── RequestedAsset (what building/system needs work)
├── WorkScope (what needs to be done)
├── Priority (CRITICAL, HIGH, NORMAL, LOW)
├── Assignment (who is doing work: staff or contractor)
├── ScheduledDate (when work will occur)
├── ActualStartDate (when work actually started)
├── CompletionDate (when work completed)
├── VerificationRecord (who verified; how verified)
├── CostTracking (labor hours, materials, contractor invoice)
└── AuditLog (all decisions and actions)
```

### Entity Definitions

#### CASE

| Attribute | Type | Description |
|---|---|---|
| CaseID | UUID | Immutable unique identifier |
| CaseType | String | Always "FIX" |
| RequestType | Enum | WorkOrder, MaintenanceRequest, CapitalProject |
| RequestedAsset | String | Building/system needing work (e.g., "Roof - Town Hall") |
| WorkScope | String | Description of what needs to be done |
| Priority | Enum | CRITICAL, HIGH, NORMAL, LOW |
| SkillRequired | String | Electrical, Plumbing, HVAC, Carpentry, IT, General |
| Requester | Object | Who submitted; contact info |
| AssignedTo | String | Technician or contractor name |
| ScheduledDate | Date | When work scheduled |
| ActualStartDate | Date (nullable) | When work actually started |
| CompletionDate | Date (nullable) | When work actually completed |
| VerificationStatus | Enum | NOT_VERIFIED, VERIFIED, FAILED |
| VerifiedBy | String | Who verified completion |
| VerificationMethod | String | Inspection, testing, requester confirmation, etc. |
| EstimatedCost | Decimal | Budget estimate |
| ActualCost | Decimal (nullable) | Actual cost after completion |
| AssignmentStatus | Enum | UNASSIGNED, ASSIGNED, IN_PROGRESS, COMPLETED |

---

## Part IV: Critical Gates

### Gate 1: Priority and SLA Assignment

**When:** INTAKE state
**Condition:** Priority assigned based on work scope
**Action:** Set response SLA and completion SLA per priority table

---

### Gate 2: Skill Matching

**When:** TRIAGE state
**Condition:** Identify required skill; determine if in-house or contractor
**Action:** Route to appropriate assignment queue

---

### Gate 3: Scheduling Confirmation

**When:** SCHEDULING state
**Condition:** Technician/contractor confirmed available
**Action:** Set ScheduledDate; notify requester

---

### Gate 4: Completion Verification

**When:** COMPLETION_REVIEW state
**Condition:** Work completed per specification; verified by authorized person
**Pass:** Lock verification record; proceed to closure
**Fail:** Work incomplete; return to IN_PROGRESS or escalate

---

### Gate 5: Cost Reconciliation

**When:** CLOSURE state
**Condition:** Actual cost documented and reconciled against estimate
**Action:** Lock cost record; post to GL (if applicable)

---

## Part V: Implementation Boundaries

### PublicLogic™ Responsibility

- Framework definition and governance authority
- Domain model and entity definitions
- Priority classification and SLA standards
- Verification requirements and process rules

### Runtime Partner Responsibility (Polimorphic)

- State machine implementation
- Data schema implementation
- Priority SLA tracking and alerting
- Assignment queue management
- Verification workflow enforcement
- Cost tracking integration

### Municipal Authority Responsibility

- Technician/contractor designation and contact
- Skill classification (who does what)
- Verification authority (who can sign off)
- GL code assignment (for cost posting)

---

## Part VI: Compliance Requirements

### Audit Trail Requirements

Every FIX CASE must maintain immutable audit log capturing:

- Request intake and priority assignment
- All assignment and scheduling decisions
- Work start and completion events
- Verification completion and sign-off
- Cost tracking and reconciliation
- Any SLA misses or escalations

### Retention Requirements

| Record Type | Minimum Retention |
|---|---|
| CASE container | 3 years from closure |
| WorkCompletion Verification | 3 years (or per capital asset policy) |
| ContractorInvoice | 7 years (per financial policy) |
| Audit log | 3 years from closure |

---

*This document is the authoritative governance architecture for VAULT FIX™. Implementation must conform to the companion Encoding Specification.*
