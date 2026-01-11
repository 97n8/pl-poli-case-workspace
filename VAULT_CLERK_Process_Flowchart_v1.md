# VAULT CLERK™ — Process Flowchart

**State Machine Visualization • Version 1.0**

---

## CLERK Module State Machine (Mermaid)

```mermaid
graph TD
    A[INTAKE] -->|Receipt recorded| B[COMPLETENESS_CHECK]
    B -->|All required docs| C[ASSESSMENT]
    B -->|Missing docs| D[REQUEST_CLARIFICATION]
    D -->|Docs provided| C
    
    C -->|InspectionRequired=true| E[INSPECTION_SCHEDULED]
    C -->|InspectionRequired=false| F[APPROVAL_REVIEW]
    
    E -->|Inspector assigned| G[INSPECTION_PENDING]
    G -->|Inspection completed| H[INSPECTION_REVIEW]
    
    H -->|PASS| F
    H -->|FAIL/CONDITIONAL| I[REQUEST_CORRECTIONS]
    I -->|Corrections submitted| H
    
    F -->|Approval authority reviews| J[APPROVAL_PENDING]
    J -->|Authority approves| K[APPROVED]
    J -->|Authority denies| L[DENIED]
    J -->|T_DECISION deadline missed| M[DEEMED_APPROVED]
    
    K -->|License issued| N[CLOSED]
    L -->|Denial final| N
    M -->|Auto-approval recorded| N
    
    N -->|Case locked| O[LOCKED]
    
    style A fill:#e1f5ff
    style N fill:#c8e6c9
    style O fill:#ffccbc
    style L fill:#ffcdd2
    style M fill:#fff9c4
```

---

## Gate Decision Points

### Gate: Completeness (B → C or D)

```mermaid
graph LR
    A{All required docs<br/>for license type<br/>present?}
    A -->|YES| B[ASSESSMENT]
    A -->|NO| C[REQUEST CLARIFICATION<br/>Clock tolled]
    C -->|Docs provided| B
```

### Gate: Inspection Required (C → E or F)

```mermaid
graph LR
    A{Inspection<br/>required for<br/>this license<br/>type?}
    A -->|YES| B[SCHEDULE INSPECTION<br/>Set T_INSPECTION]
    A -->|NO| C[SKIP TO APPROVAL<br/>No inspection needed]
```

### Gate: Inspection Results (H → F or I)

```mermaid
graph LR
    A{Inspection<br/>result?}
    A -->|PASS| B[APPROVAL REVIEW]
    A -->|FAIL/CONDITIONAL| C[REQUEST CORRECTIONS<br/>Clock tolled]
    C -->|Resubmit| A
```

### Gate: Approval Decision (J → K, L, or M)

```mermaid
graph LR
    A{Approval<br/>decision made<br/>before T_DECISION<br/>deadline?}
    A -->|APPROVE| B[APPROVED<br/>Lock letter]
    A -->|DENY| C[DENIED<br/>Lock letter]
    A -->|MISSED DEADLINE| D[DEEMED APPROVED<br/>Auto-generated]
```

---

## Timeline Example: Contractor License

```
Day 0 (Jan 15): Application submitted
  └─ INTAKE → COMPLETENESS_CHECK
     ✓ Insurance document
     ✓ Bonding document
     ✓ Background check form
  
Day 1 (Jan 16): Completeness verified
  └─ COMPLETENESS_CHECK → ASSESSMENT
  
Day 2 (Jan 17): Inspection scheduled
  └─ ASSESSMENT → INSPECTION_SCHEDULED → INSPECTION_PENDING
     Inspector: Building Inspector Jones
     Date: Jan 19
  
Day 4 (Jan 19): Inspection conducted
  └─ INSPECTION_PENDING → INSPECTION_REVIEW
     Result: PASS
  
Day 5 (Jan 20): Approval reviewed
  └─ INSPECTION_REVIEW → APPROVAL_REVIEW → APPROVED
     Authority: Building Inspector Jones
     Approval recorded and LOCKED
  
Day 5 (Jan 20): License issued
  └─ APPROVED → CLOSED
     License Certificate generated (LOCKED)
     LicenseExpiry: Jan 20, 2027 (1 year)
     
T90 Window Opens: Jan 20 - Feb 19 (30 calendar days)
  Applicant can appeal if desired
```

---

## Error Paths

```
If T_INTAKE missed:
  INTAKE → REQUEST_CLARIFICATION (hard stop at deadline)
  
If T_DECISION missed:
  APPROVAL_REVIEW → DEEMED_APPROVED (automatic)
  ApprovalStatus = APPROVED
  License issued automatically
  
If Inspection fails:
  INSPECTION_REVIEW → REQUEST_CORRECTIONS (tolled)
  Clock paused until corrections submitted + re-inspection
```

---

*Process flowchart shows all states, transitions, gates, and conditions.*
