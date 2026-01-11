# VAULT Module System Architectures

**Three-Rail Architecture Diagrams • Version 1.0**

---

## Architecture Pattern: Three Rails

Every VAULT module follows the same three-rail architecture:

1. **Left Rail (LEGEND)** — State definitions, entity definitions, schema
2. **Center Rail (CONTROL SPINE)** — State machine, transitions, gates, timers
3. **Right Rail (DOMAIN MODEL)** — Entities, assets, audit log, retention

---

## CLERK Module — Three-Rail Architecture

```
LEFT RAIL (LEGEND)          CENTER RAIL (CONTROL)       RIGHT RAIL (DOMAIN)
─────────────────────       ────────────────────        ───────────────────

CASE States:                 CASE Lifecycle:             CASE Entity:
  INTAKE                       INTAKE →                   CaseID (UUID)
  COMPLETENESS_CHECK           COMPLETENESS_CHECK →       CaseType: CLERK
  ASSESSMENT                   ASSESSMENT →               ApplicationType
  INSPECTION_SCHEDULED         INSPECTION_SCHEDULED →     Applicant
  INSPECTION_PENDING           INSPECTION_PENDING →       Deadlines (T_*)
  INSPECTION_REVIEW            INSPECTION_REVIEW →        ApprovalStatus
  APPROVAL_REVIEW              APPROVAL_REVIEW →          Assets (array)
  APPROVED                     APPROVED →                 AuditLog (array)
  DENIED                       DENIED →
  CLOSED                       CLOSED

Timers:                      Gate Enforcement:          Assets:
  T_INTAKE (2 days)           Completeness check         Application (REF)
  T_DECISION (15-20 days)     Inspection required?       InspectionReport (KEEPER)
  T_INSPECTION (5-10 days)    Inspection results         ApprovalLetter (KEEPER)
  T90 (30 calendar days)      Approval authority         DenialLetter (KEEPER)
                                                         License Cert (KEEPER)

Retention:                   Rules:
  KEEPER: Permanent            Deemed approval on deadline miss
  REFERENCE: 7 years           No edits after lock
  TRANSACTIONAL: 1-2 years     Segregation of duties (if applicable)
                                
                              Audit Log:
                                All state transitions
                                All asset locks
                                All deadline enforcements
                                All gate results
```

---

## FISCAL Module — Three-Rail Architecture

```
LEFT RAIL (LEGEND)          CENTER RAIL (CONTROL)       RIGHT RAIL (DOMAIN)
─────────────────────       ────────────────────        ───────────────────

CASE States:                 CASE Lifecycle:             CASE Entity:
  INTAKE                       INTAKE →                   CaseID (UUID)
  VALIDATION                   VALIDATION →               CaseType: FISCAL
  MATCHING                     MATCHING →                 Vendor
  VARIANCE_REVIEW              VARIANCE_REVIEW →          Invoice (object)
  BUDGET_CHECK                 BUDGET_CHECK →             PO (linked)
  APPROVAL_ROUTING             APPROVAL_ROUTING →         Receipt (linked)
  APPROVAL_PENDING             APPROVAL_PENDING →         Matching (object)
  APPROVED                     APPROVED →                 Budget (object)
  PAYMENT_PROCESSING           PAYMENT_PROCESSING →       Approval (object)
  PAYMENT_COMPLETE             PAYMENT_COMPLETE →         Payment (object)
  RECONCILIATION               RECONCILIATION →           GLPosting (object)
  CLOSED                       CLOSED                     AuditLog (array)

Timers:                      Gate Enforcement:          Assets:
  T_PAYMENT_DUE (Net 30)      3-way match validation    Invoice (REFERENCE)
  T_APPROVAL (5 days)         Budget availability       PO (KEEPER)
  T_RECONCILIATION (30 days)  Vendor authorization      Receipt (KEEPER)
                              Approval routing per $     ApprovalSignature (KEEPER)
Retention:                      Duplicate detection       Payment Confirmation (KEEPER)
  KEEPER: 7 years                                        GL Posting (KEEPER)
  REFERENCE: 7 years          Rules:
  TRANSACTIONAL: 2 years        Approval signatures LOCKED
                                GL posting LOCKED
                                3-way match mandatory
                                No payment without approval

                              Audit Log:
                                Matching results
                                Variance explanations
                                Approval decisions (immutable)
                                Payment execution
                                GL posting & reconciliation
```

---

## FIX Module — Three-Rail Architecture

```
LEFT RAIL (LEGEND)          CENTER RAIL (CONTROL)       RIGHT RAIL (DOMAIN)
─────────────────────       ────────────────────        ───────────────────

CASE States:                 CASE Lifecycle:             CASE Entity:
  INTAKE                       INTAKE →                   CaseID (UUID)
  TRIAGE                       TRIAGE →                   CaseType: FIX
  SCHEDULING                   SCHEDULING →               RequestType
  IN_PROGRESS                  IN_PROGRESS →              RequestedAsset
  COMPLETION_REVIEW            COMPLETION_REVIEW →        WorkScope
  CLOSURE                      CLOSURE                    Priority (SLA)
  CLOSED                                                  SkillRequired
                                                          Requester
Priority SLAs:               Gate Enforcement:          AuditLog (array)
  CRITICAL: 2h response        Skill identification
  HIGH: 4h response            Assignment confirmation
  NORMAL: 24h response         SLA monitoring
  LOW: 5d response             Completion verification
                                
  Completion SLAs:           Rules:
  CRITICAL: 24h                Verification before closure
  HIGH: 5 days                 SLA tracking (soft, not hard)
  NORMAL: 14 days              Verification LOCKED
  LOW: 30 days

Verification Methods:        Audit Log:
  Inspection + sign-off        Priority & SLA assignment
  Testing + confirmation       Assignment changes
  Requester sign-off           SLA compliance/miss
                               Completion verification
                               Cost tracking
```

---

## TIME Module — Three-Rail Architecture

```
LEFT RAIL (LEGEND)          CENTER RAIL (CONTROL)       RIGHT RAIL (DOMAIN)
─────────────────────       ────────────────────        ───────────────────

CASE States:                 CASE Lifecycle:             CASE Entity:
  ENTRY                        ENTRY →                    CaseID (UUID)
  REVIEW                       REVIEW →                   CaseType: TIME
  APPROVAL                     APPROVAL →                 PayPeriod
  PAYROLL_POSTING              PAYROLL_POSTING →          Employee
  CLOSED                       CLOSED → LOCKED            HoursDetails
                                                          Allocations[]
Timers (Soft SLAs):          Gate Enforcement:          ApprovalStatus
  T_ENTRY: Friday EOD          Completeness check        Supervisor Approval
  T_APPROVAL: Monday 9 AM      Variance explanation      Finance Approval (if OT)
  T_PAYROLL: Tuesday 5 PM      Overtime authorization    PayrollPostingDate
                               Segregation of duties     IsLocked (bool)
Overtime Rules:                                          AuditLog (array)
  0-40 hrs: Supervisor only   Rules:
  40-48 hrs: Finance approval   No edits after posting
  48+ hrs: Director approval    Timesheet LOCKED at payroll
                                Budget impact for OT

                              Audit Log:
                                Entry & submission
                                Manager review
                                Overtime flagging
                                Approvals (immutable)
                                Payroll posting
                                GL allocation
```

---

## ONBOARD Module — Three-Rail Architecture

```
LEFT RAIL (LEGEND)          CENTER RAIL (CONTROL)       RIGHT RAIL (DOMAIN)
─────────────────────       ────────────────────        ───────────────────

CASE States:                 CASE Lifecycle:             CASE Entity:
  INTAKE                       INTAKE →                   CaseID (UUID)
  PRE_START                    PRE_START →                CaseType: ONBOARD
  DAY_ONE                      DAY_ONE →                  Employee (object)
  SYSTEMS_SETUP                SYSTEMS_SETUP →            BackgroundCheck
  TRAINING                     TRAINING →                 PreStartTasks
  READINESS_REVIEW             READINESS_REVIEW →         DayOneTasks
  CLOSED                       CLOSED → LOCKED            SystemsSetupTasks
                                                          TrainingTasks (LOCKED)
Tasks (Mandatory):           Gate Enforcement:          Manager SignOff (LOCKED)
  Background check clear       Background cleared?       AuditLog (array)
  I-9 completed                All tasks documented?
  Email created                All training locked?
  Network access               Manager confirmed ready?
  Policy training (LOCKED)
  Department training (LOCKED) Rules:
  Manager sign-off (LOCKED)    No start without background clear
                               All training must be locked
Retention:                      Manager sign-off mandatory
  KEEPER: Permanent            No skipping any task
  (Offer, BGC, I9, Training,
   SignOff)

                              Audit Log:
                                Task completion dates
                                Background check results
                                Training completion & locks
                                Manager sign-off
                                All timestamps (immutable)
```

---

## Cross-Module Pattern Recognition

All VAULT modules follow same three-rail pattern:

1. **LEFT RAIL** defines what can exist
2. **CENTER RAIL** enforces what must happen
3. **RIGHT RAIL** records what did happen

Variations:
- **PRR & CLERK**: Hard deadline enforcement (T10, T_DECISION, etc.)
- **FISCAL**: Approval gates + budget checks
- **FIX**: SLA response tracking + completion verification
- **TIME**: Soft SLAs + overtime routing
- **ONBOARD**: Task completion gates + no enforcement

All have:
- Immutable audit logs
- LOCKED assets (cannot edit)
- Version chains (if correction needed)
- Asset retention classes (KEEPER, REFERENCE, TRANSACTIONAL)

---

*Three-rail architecture is standard across all VAULT modules.*
