# VAULT Module Process Flowcharts

**Visual State Machine Diagrams • Version 1.0**

---

## FISCAL Module Flowchart

```mermaid
graph TD
    A[INTAKE] -->|Invoice received| B[VALIDATION]
    B -->|All fields present| C[MATCHING]
    B -->|Incomplete| A
    
    C -->|3-way match| D{Match<br/>result?}
    D -->|MATCHED| E[BUDGET_CHECK]
    D -->|VARIANCE| F[VARIANCE_REVIEW]
    D -->|NO_PO/NO_RECEIPT| G[REQUEST_DOCUMENTS]
    
    F -->|Explained & approved| E
    G -->|Documents received| C
    
    E -->|Budget available| H[APPROVAL_ROUTING]
    E -->|Insufficient| I[ESCALATE_BUDGET]
    I -->|Budget approved| H
    
    H -->|Route to authority| J[APPROVAL_PENDING]
    J -->|Approved| K[PAYMENT_PROCESSING]
    J -->|Denied| A
    
    K -->|Payment sent| L[PAYMENT_COMPLETE]
    L -->|Bank confirmation| M[RECONCILIATION]
    M -->|Reconciled| N[CLOSED]
    
    style A fill:#e1f5ff
    style N fill:#c8e6c9
    style K fill:#b3e5fc
    style J fill:#fff9c4
```

---

## FIX Module Flowchart

```mermaid
graph TD
    A[INTAKE] -->|Request received| B[TRIAGE]
    B -->|Assess| C{Priority<br/>assignment}
    
    C -->|CRITICAL| D["Set SLA: 2h response, 24h completion"]
    C -->|HIGH| E["Set SLA: 4h response, 5d completion"]
    C -->|NORMAL| F["Set SLA: 24h response, 14d completion"]
    C -->|LOW| G["Set SLA: 5d response, 30d completion"]
    
    D --> H[SCHEDULING]
    E --> H
    F --> H
    G --> H
    
    H -->|Assign technician| I[IN_PROGRESS]
    I -->|Work completed| J[COMPLETION_REVIEW]
    
    J -->|Verified by authority| K{Verification<br/>passed?}
    K -->|YES| L[CLOSURE]
    K -->|NO| M[RETURN_TO_WORK]
    M -->|Rework completed| J
    
    L -->|Cost reconciled| N[CLOSED]
    
    style A fill:#e1f5ff
    style N fill:#c8e6c9
    style I fill:#b3e5fc
    style J fill:#fff9c4
```

---

## TIME Module Flowchart

```mermaid
graph TD
    A[ENTRY] -->|Employee enters hours| B[REVIEW]
    B -->|Manager reviews| C{Accuracy<br/>check}
    
    C -->|OK| D{Hours > 40<br/>per week?}
    C -->|Clarification needed| E[REQUEST_CLARIFICATION]
    E -->|Explanation provided| D
    
    D -->|NO| F[APPROVAL]
    D -->|YES| G[OVERTIME_AUTHORIZATION]
    G -->|Finance approves| F
    G -->|Denied| E
    
    F -->|Manager approves| H[PAYROLL_POSTING]
    H -->|Posted to payroll| I[LOCKED]
    I -->|Timesheet immutable| J[CLOSED]
    
    style A fill:#e1f5ff
    style J fill:#c8e6c9
    style H fill:#b3e5fc
    style F fill:#fff9c4
```

---

## ONBOARD Module Flowchart

```mermaid
graph TD
    A[INTAKE] -->|Offer accepted| B[PRE_START]
    B -->|Background ordered| C{Background<br/>cleared?}
    
    C -->|NO| D[ESCALATE_HR]
    D -->|Not cleared, do not proceed| E["DO NOT START"]
    
    C -->|YES| F[DAY_ONE]
    F -->|I-9, badge, welcome| G[SYSTEMS_SETUP]
    G -->|Email, network, systems| H[TRAINING]
    
    H -->|HR handbook| I[TRAINING_LOCKED_1]
    H -->|Safety briefing| J[TRAINING_LOCKED_2]
    H -->|Compliance training| K[TRAINING_LOCKED_3]
    H -->|Department training| L[TRAINING_LOCKED_4]
    
    I --> M[READINESS_REVIEW]
    J --> M
    K --> M
    L --> M
    
    M -->|Manager confirms ready| N[MANAGER_SIGNOFF]
    N -->|Sign-off locked| O[CLOSED]
    
    style A fill:#e1f5ff
    style O fill:#c8e6c9
    style F fill:#b3e5fc
    style M fill:#fff9c4
    style E fill:#ffcdd2
```

---

*All module process flowcharts with gate decisions and state transitions.*
