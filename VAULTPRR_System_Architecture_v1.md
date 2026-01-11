# VAULTPRR™ — System Architecture Diagram

**Canonical Domain Model & Control Spine**  
**Version 1.0**

---

```mermaid
flowchart LR
%% ============================================================
%% VAULTPRR™ — Canonical System Architecture & Domain Model
%% Framework-as-a-Standard™ • v1.0 • Massachusetts PRR
%% Authority: M.G.L. c. 66, §10 • 950 CMR 32.00
%% ============================================================

%% ---------------------------
%% LEFT RAIL — LEGEND / SEMANTICS
%% ---------------------------
subgraph L["Legend & Semantics"]
direction TB

subgraph LT["Step Types"]
L1["FormStep"]
L2["SystemEvent"]
L3["ReviewStep"]
L4["ManualStep"]
L5["DocumentStep"]
L6["SendEmailStep"]
L7["PaymentStep"]
L8["DecisionGate"]
L9["WaitState (no mutation)"]
end

subgraph LC["Record Classes"]
LC1["KEEPER: Permanent"]
LC2["Reference: 7 years"]
LC3["Transactional: 1-2 years"]
end

subgraph LM["Immutability"]
LM1["LOCKED Artifact"]
LM2["Versioned Replacement Only"]
end

subgraph LD["Statutory Timers"]
LD1["T10 Initial Response"]
LD2["T20 Petition Window"]
LD3["T25 Municipal Limit"]
LD4["T90 Appeal Window"]
end

subgraph LS["Implementation Stack"]
LS1["PublicLogic — Framework"]
LS2["Polimorphic — Runtime"]
LS3["PublicInsight — Training"]
end
end

%% ---------------------------
%% CENTER — ABSTRACT CONTROL SPINE
%% ---------------------------
subgraph C["Abstract Control Spine"]
direction TB
S0((START)) --> S1(( )) --> S2(( )) --> S3(( )) --> S4(( )) --> S5(( )) --> S6(( )) --> S7(( )) --> S8((END))
S3 -.->|Clarify| S2
S3 -.->|No Records| S8
S5 -.->|Withhold| S8
S7 -.->|Non-Payment| S8
end

%% ---------------------------
%% RIGHT RAIL — DOMAIN MODEL
%% ---------------------------
subgraph R["Domain Model"]
direction TB

subgraph CASE["CASE Workspace"]
RC1["CaseID: UUID"]
RC2["RequesterIdentity"]
RC3["IntakeChannel"]
RC4["Scope (versioned)"]
RC5["EffectiveReceiptDate"]
RC6["DeadlineModel (T10/T20/T25/T90)"]
RC7["FeesAllowed"]
RC8["Status / ClosureReason"]
end

subgraph PROCESS["Process"]
RP1["ProcessID"]
RP2["ActiveStepID"]
RP3["EnforcementRules"]
end

subgraph STEP["Step"]
RS1["StepID"]
RS2["StepType"]
RS3["KeeperClass"]
RS4["Locked"]
RS5["RequiredCapture"]
end

subgraph PATH["Path"]
RPA1["From / To"]
RPA2["Condition"]
RPA3["TollRule"]
RPA4["GateRule"]
end

subgraph ASSET["Asset"]
RA1["AssetID"]
RA2["AssetType"]
RA3["RetentionPolicy"]
RA4["Locked"]
RA5["VersionChain"]
end

subgraph LOG["Log"]
RL1["Timestamp"]
RL2["Actor"]
RL3["Action"]
RL4["AssetsAffected"]
end
end

%% ---------------------------
%% LIFECYCLE ATTACHMENT
%% ---------------------------
S1 --- I["Intake (FormStep)<br/>KEEPER"]
S2 --- E["Compute Timers (SystemEvent)<br/>T10/T20/T25"]
S3 --- A["RAO Assessment (ReviewStep)<br/>KEEPER"]
S4 --- G["Gather / Search (ManualStep)<br/>KEEPER"]
S5 --- X["Exemption Review (ReviewStep)<br/>KEEPER"]
S6 --- P["Response Package (DocumentStep)<br/>LOCKED"]
S7 --- D["Fees / Payment / Delivery"]

A -.-> CL["Clarification Email"]
CL -.-> W["Wait"]
W -.-> T["Toll & Recompute"]
T -.-> A

A -.-> NR["No Records Letter<br/>LOCKED"]
X -.-> WD["Withholding Letter<br/>LOCKED"]

P --> T10G{"T10 Met?"}
T10G -->|No| WAIVE["FeesAllowed = false"]
WAIVE --> DEL["Deliver Records"]
T10G -->|Yes| FE{"Fees Required?"}
FE -->|No| DEL
FE -->|Yes| INV["Invoice"]
INV --> PAY{"Paid?"}
PAY -->|Yes| DEL
PAY -->|No| NP["Close: Non-Payment"]

DEL --> ENDOK["Close: Delivered"]

A --> FC{"Forecast > T25?"}
FC -->|No| G
FC -->|Yes & agreement| AGR["Requester Agreement"]
AGR --> G
FC -->|Yes & no agreement| PET["Petition"]
PET --> G
```

---

## Reading This Diagram

**Three Rails:**

| Rail | Purpose |
|------|---------|
| **Left: Legend** | Vocabulary — step types, record classes, timers, implementation stack |
| **Center: Control Spine** | Abstract process flow — states and transitions |
| **Right: Domain Model** | Data structures — CASE, Process, Step, Path, Asset, Log |

**Lifecycle Attachment:** The center spine connects to concrete steps (Intake, Assessment, Gather, etc.) showing where domain entities are created and modified.

**Use this diagram for:** Understanding the system architecture — what entities exist, how they relate, what the control flow looks like at an abstract level.

**Use the Process Flowchart for:** Implementation — concrete states, transitions, decision gates, and validation rules.
