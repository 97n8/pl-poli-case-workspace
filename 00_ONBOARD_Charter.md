# VAULT ONBOARD Module — Charter

**Employee Onboarding and Institutional Transfer**

---

## Module Purpose

VAULT ONBOARD ensures employee onboarding is fully documented and transferable.

When a new employee arrives, a CASE is created. Every step of the onboarding process is recorded and locked.

When the employee leaves, their CASE shows everything that was done. Replacement hire can follow the exact same path.

---

## CASE Model for ONBOARD

### Identity

```json
{
  "CaseID": "onboard-2026-emp-001",
  "CaseType": "ONBOARD",
  "EmployeeID": "EMP-001",
  "StartDate": "2026-03-01"
}
```

### Subject

```json
{
  "EmployeeIdentity": {
    "Name": "Alice Johnson",
    "Role": "Administrative Assistant",
    "Department": "Finance",
    "Supervisor": "Bob Smith"
  }
}
```

### Scope

```json
{
  "ScopeDefinition": {
    "OnboardingTasks": [
      "IT Account Setup",
      "Badge and Building Access",
      "Equipment Assignment",
      "Policy Training",
      "System Access Provisioning",
      "Department-Specific Training"
    ]
  }
}
```

### Assets

```json
{
  "Assets": [
    {"AssetType": "OfferLetter", "RetentionClass": "KEEPER"},
    {"AssetType": "ITAccountSetup", "RetentionClass": "REFERENCE"},
    {"AssetType": "BadgeAndAccess", "RetentionClass": "REFERENCE"},
    {"AssetType": "EquipmentAssignment", "RetentionClass": "REFERENCE"},
    {"AssetType": "TrainingCompletion", "RetentionClass": "KEEPER"},
    {"AssetType": "OnboardingCertificate", "RetentionClass": "KEEPER", "Locked": true}
  ]
}
```

---

## Module-Specific Rules

### No Deadlines

Unlike PRR or FISCAL, ONBOARD has no statutory deadlines. It is a single-pass process.

Ideally completed within 30 days of start, but no hard deadline.

### Task Interdependencies

Some tasks must happen before others:
* IT account before system access
* Badge before building access
* Policy training before system use

CASE tracks dependencies and order.

### Transferability

Core goal: When employee leaves, replacement can see exactly what was done.

Example: New assistant starts. CASE shows:
* Jane was given access to Finance system on Day 2
* Jane was given badge access to Building A on Day 1
* Jane completed Financial Procedures training on Day 3

New assistant follows same path.

---

## Process Flow (High Level)

```
Offer → IT Setup → Access → Equipment → Training → Completion → Closure
```

---

## Asset Types in ONBOARD

| Asset | Retention | Locked? | Purpose |
|-------|-----------|---------|---------|
| OfferLetter | KEEPER | Yes | Employment confirmation |
| ITAccountSetup | REFERENCE | No | Credentials created |
| BadgeAndAccess | REFERENCE | No | Physical access provisioned |
| EquipmentAssignment | REFERENCE | No | Laptop, phone, etc. |
| TrainingRecords | KEEPER | No | What training was completed |
| OnboardingCertificate | KEEPER | Yes | Onboarding complete |

---

## Onboarding Task Checklist

### Pre-Start (Before start date)

- [ ] Background check ordered and cleared
- [ ] Background check result recorded (KEEPER)
- [ ] I-9 documentation collected
- [ ] Emergency contact form collected
- [ ] Position description prepared
- [ ] Workspace assigned and prepared

### Day 1 (Start date)

- [ ] IT account created (credentials)
- [ ] Badge/access card issued
- [ ] Building access provisioned
- [ ] Welcome packet provided
- [ ] Supervisor introduction
- [ ] Department orientation
- [ ] Safety briefing (if applicable)

### Days 2-5

- [ ] System access provisioned (email, VPN, databases, etc.)
- [ ] Equipment assigned (laptop, phone, etc.)
- [ ] Policy training (HR handbook, safety, compliance)
- [ ] Department-specific training begins
- [ ] Project assignment communicated

### Week 2-4

- [ ] Department training completed
- [ ] System proficiency verified (can perform core tasks)
- [ ] Manager check-in: progress review
- [ ] Completion of all required training (LOCKED)
- [ ] Onboarding sign-off document

---

## Task Dependencies

```
Background Check
    ↓ (must clear before start)
I-9 Documentation
    ↓ (collected before start)
Day 1: IT Account ← (needed before Day 2 system access)
    ↓
Day 2: System Access ← (requires IT account)
    ↓
Day 3-5: Training (can start immediately)
    ↓
Week 3: Proficiency Check ← (after training)
    ↓
Onboarding Complete ← [LOCKED]
```

---

## Process Flow (High-Level)

```
[Offer Accepted]
  ↓
[Pre-Start Tasks]
  ├─ Background check ordered
  ├─ IT account creation scheduled
  ├─ Workspace preparation
  └─ All locked in (KEEPER records)
  ↓
[Day 1: Arrival]
  ├─ Welcome & orientation
  ├─ Badge issued
  ├─ Initial briefings
  ↓
[Days 2-5: System Setup & Training]
  ├─ IT systems access (email, VPN, etc.)
  ├─ Equipment provisioned
  ├─ Policy training
  ├─ Department orientation
  ↓
[Week 2-4: Department Training]
  ├─ Role-specific training
  ├─ System proficiency
  ├─ Manager check-in
  ↓
[Completion Verification]
  ├─ All tasks checked off
  ├─ Training documented (LOCKED)
  ├─ Manager sign-off (LOCKED)
  ↓
[Onboarding Certificate] → [LOCKED]
  ↓
[Closed]
```

---

## Compliance Checklist for ONBOARD CASE

Before closing:

- [ ] Offer letter recorded (KEEPER)
- [ ] Background check completed and cleared
- [ ] I-9 documentation complete
- [ ] IT account created with credentials
- [ ] Badge and building access granted
- [ ] Equipment assigned (laptop, phone, etc.)
- [ ] Email and system access working
- [ ] All required training completed (KEEPER)
- [ ] Manager confirms readiness (LOCKED)
- [ ] Onboarding certificate generated (LOCKED)
- [ ] All documents recorded (KEEPER)
- [ ] Audit log sealed
- [ ] Employee is ready to work independently

---

## Module Expansion Notes

Detailed implementation will include:
* Department-specific onboarding checklists (Finance vs. IT vs. Public Works)
* Compliance training matrix (OSHA, HIPAA, etc.)
* System access request workflows
* Equipment provisioning integration
* Contractor vs. permanent employee differences
* Rehire vs. new hire workflows

---

*VAULT ONBOARD follows the same CASE architecture. Its purpose is institutional transfer, not enforcement of statute. Simplest of all VAULT modules.*
