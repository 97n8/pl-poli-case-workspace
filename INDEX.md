# VAULT Master Engineering Packet v1.0 — Index and Navigation

**Complete Framework-as-a-Standard™ for Municipal Governance**

---

## Quick Start (30 Minutes)

1. **Read:** `00_README/Read_First.md` (5 min)
2. **Understand:** `00_README/Audience_and_Use.md` (5 min)  
3. **Know Limits:** `00_README/What_Is_Out_of_Scope.md` (5 min)
4. **Learn Core:** `01_VAULT_CORE_CANON/CASE_Workspace.md` (15 min)

After this, you should understand what VAULT is and whether it fits your needs.

---

## Full Orientation (2-3 Hours)

### Foundation (1 hour)

Read these in order:

1. `00_README/Read_First.md` — What this packet is
2. `00_README/Audience_and_Use.md` — Who it's for
3. `01_VAULT_CORE_CANON/Authority_and_Scope.md` — Why it exists
4. `01_VAULT_CORE_CANON/CASE_Workspace.md` — The root entity

### Architecture (1 hour)

Then read these to understand how VAULT works:

1. `01_VAULT_CORE_CANON/Record_Classes.md` — Data classification
2. `01_VAULT_CORE_CANON/Timer_and_Deadline_Model.md` — How deadlines work
3. `01_VAULT_CORE_CANON/Audit_and_Evidence_Model.md` — How trust is maintained

### Reference Implementation (30 min)

See how it all works in practice:

1. `02_MODULES/PRR/00_PRR_Charter.md` — Public records reference

### Implementation Realities (30 min)

Before you code:

1. `03_IMPLEMENTATION_GUIDANCE/What_Engineers_May_Choose.md` — Where you have flexibility
2. `03_IMPLEMENTATION_GUIDANCE/What_Is_Non_Negotiable.md` — Where you don't

---

## By Role

### If You Are a Governance/Legal Person

**Read first:**
* `00_README/Read_First.md`
* `01_VAULT_CORE_CANON/Authority_and_Scope.md`
* `01_VAULT_CORE_CANON/Timer_and_Deadline_Model.md`

**Then module charters for relevant modules:**
* `02_MODULES/PRR/00_PRR_Charter.md` (if public records)
* `02_MODULES/CLERK/00_CLERK_Charter.md` (if licensing)
* Etc.

**Reference:**
* `99_APPENDICES/Glossary.md`

### If You Are an Engineer / Architect

**Start with:**
* `00_README/Read_First.md`
* `00_README/Audience_and_Use.md`
* `01_VAULT_CORE_CANON/CASE_Workspace.md` (critical)

**Then deep dive:**
* All of `01_VAULT_CORE_CANON/` (2-3 hours)
* `02_MODULES/PRR/` (reference implementation)
* `03_IMPLEMENTATION_GUIDANCE/What_Engineers_May_Choose.md`
* `03_IMPLEMENTATION_GUIDANCE/What_Is_Non_Negotiable.md`

**Return to for reference:**
* `99_APPENDICES/Glossary.md`
* `99_APPENDICES/VERSION_HISTORY.md`

### If You Are an Operations / Support Person

**Start with:**
* `00_README/Read_First.md`
* Relevant module charter (e.g., `02_MODULES/PRR/00_PRR_Charter.md`)

**Then:**
* `01_VAULT_CORE_CANON/CASE_Workspace.md` (understand the container)
* `99_APPENDICES/Glossary.md` (reference terminology)

---

## By Module

Each module has the same structure. Start with the charter:

### VAULTPRR (Public Records)

* `02_MODULES/PRR/00_PRR_Charter.md` — Overview
* `02_MODULES/PRR/Control_Flow.mmd` — Detailed process (Mermaid diagram)
* `02_MODULES/PRR/Decision_Tables.md` — Gates and outcomes
* `02_MODULES/PRR/Artifacts.md` — Asset types and retention

### VAULT CLERK (Licensing)

* `02_MODULES/CLERK/00_CLERK_Charter.md` — Overview
* Add more as you need detail

### VAULT FISCAL (Invoices and Procurement)

* `02_MODULES/FISCAL/00_FISCAL_Charter.md` — Overview

### VAULT TIME (Timesheets)

* `02_MODULES/TIME/00_TIME_Charter.md` — Overview

### VAULT FIX (Work Orders)

* `02_MODULES/FIX/00_FIX_Charter.md` — Overview

### VAULT ONBOARD (Employee Onboarding)

* `02_MODULES/ONBOARD/00_ONBOARD_Charter.md` — Overview

---

## Directory Structure

```
VAULT_MASTER_ENGINEERING_PACKET_v1.0/
│
├── 00_README/
│   ├── Read_First.md (START HERE)
│   ├── Audience_and_Use.md
│   └── What_Is_Out_of_Scope.md
│
├── 01_VAULT_CORE_CANON/
│   ├── CASE_Workspace.md (CRITICAL)
│   ├── Authority_and_Scope.md
│   ├── Record_Classes.md
│   ├── Immutability_and_Locks.md
│   ├── Event_Model.md
│   ├── Timer_and_Deadline_Model.md
│   └── Audit_and_Evidence_Model.md
│
├── 02_MODULES/
│   ├── PRR/
│   │   ├── 00_PRR_Charter.md
│   │   ├── VAULTPRR_Governance_Architecture_v1.md
│   │   ├── VAULTPRR_Encoding_Specification_v1.md
│   │   ├── VAULTPRR_Process_Flowchart_v1.md
│   │   └── VAULTPRR_System_Architecture_v1.md
│   ├── CLERK/
│   │   ├── 00_CLERK_Charter.md
│   │   ├── VAULT_CLERK_Governance_Architecture_v1.md
│   │   ├── VAULT_CLERK_Encoding_Specification_v1.md
│   │   └── VAULT_CLERK_Process_Flowchart_v1.md
│   ├── FISCAL/
│   │   ├── 00_FISCAL_Charter.md
│   │   ├── VAULT_FISCAL_Governance_Architecture_v1.md
│   │   └── VAULT_FISCAL_Encoding_Specification_v1.md
│   ├── FIX/
│   │   ├── 00_FIX_Charter.md
│   │   ├── VAULT_FIX_Governance_Architecture_v1.md
│   │   └── VAULT_FIX_Encoding_Specification_v1.md
│   ├── TIME/
│   │   ├── 00_TIME_Charter.md
│   │   └── VAULT_TIME_Governance_Encoding_v1.md
│   ├── ONBOARD/
│   │   ├── 00_ONBOARD_Charter.md
│   │   └── VAULT_ONBOARD_Governance_Encoding_v1.md
│   ├── FLOWCHARTS_ALL_MODULES_v1.md
│   └── SYSTEM_ARCHITECTURES_ALL_MODULES_v1.md
│
├── 03_IMPLEMENTATION_GUIDANCE/
│   ├── What_Engineers_May_Choose.md
│   ├── What_Is_Non_Negotiable.md
│   ├── Testing_Validation_Checklist.md
│   ├── Error_Handling_Recovery.md
│   ├── Technology_Trade_Off_Analysis.md
│   └── Escalation_SLA.md
│
└── 99_APPENDICES/
    ├── Glossary.md
    └── VERSION_HISTORY.md
```

---

## Core Documents (Essentials)

These six documents define VAULT. Read them first:

| # | Document | Purpose |
|---|----------|---------|
| 1 | `Read_First.md` | Orientation |
| 2 | `Authority_and_Scope.md` | Why VAULT exists |
| 3 | `CASE_Workspace.md` | Root entity (critical) |
| 4 | `Record_Classes.md` | Data classification |
| 5 | `Timer_and_Deadline_Model.md` | Enforcement |
| 6 | `Audit_and_Evidence_Model.md` | Evidence trail |

Master these six, and everything else makes sense.

---

## Process Flowcharts and System Architectures

All module process flowcharts and system architecture diagrams are consolidated in:

* `02_MODULES/FLOWCHARTS_ALL_MODULES_v1.md` — All module state machines (Mermaid format)
* `02_MODULES/SYSTEM_ARCHITECTURES_ALL_MODULES_v1.md` — All three-rail architecture diagrams

Individual modules also reference these consolidated documents for visual representation of their processes.

---

## Questions During Implementation

### Governance Questions

**Example:** "Can a PRR be withdrawn after records are gathered?"

**Answer:** Look in `02_MODULES/PRR/00_PRR_Charter.md` and `99_APPENDICES/VERSION_HISTORY.md`.

If not answered: Escalate via governance gap protocol (see `00_README/Audience_and_Use.md`).

### Technical Questions

**Example:** "Should we use REST API or GraphQL?"

**Answer:** You choose. See `03_IMPLEMENTATION_GUIDANCE/What_Engineers_May_Choose.md`.

### Scope Questions

**Example:** "Should this packet include UI mockups?"

**Answer:** No. See `00_README/What_Is_Out_of_Scope.md`.

---

## Glossary

Standard VAULT vocabulary is in `99_APPENDICES/Glossary.md`.

When you see an unfamiliar term, look it up.

---

## Feedback and Contact

**Governance clarifications:** info@publiclogic.com  
**Questions:** Use escalation path in `00_README/Audience_and_Use.md`  
**Bug reports:** Document scenario and send to PublicLogic

---

## How This Packet Relates to Other Documents

**This packet (Master Engineering):**
* Governs what VAULT must do
* Authoritative for all implementations
* Stable (changes are versioned)

**Encoding Specifications (Module-specific):**
* Detail how to implement governance
* Includes JSON schemas, state machines, algorithms
* Follows from this packet

**Derivative Documents (Implementation-specific):**
* SOP, training, UI specs
* Built after this packet is locked
* Follow from Encoding Specifications

**This packet is tier 1 (governance). Encoding is tier 2 (implementation contract). Derivatives are tier 3 (operations).**

---

## Version and Authority

**Version:** 1.0 (January 2026)  
**Authority:** PublicLogic LLC  
**Status:** Authoritative  
**Confidentiality:** Internal / NDA Partners Only  

See `99_APPENDICES/VERSION_HISTORY.md` for full history and amendment process.

---

## Navigation Tips

* Use Markdown viewer with table of contents (most editors provide this)
* Search for terms in `99_APPENDICES/Glossary.md`
* Return to `Read_First.md` anytime to reorient
* Print module charters for reference during meetings

---

*You are reading the authoritative VAULT governance specification. Use it as the source of truth for all implementation decisions.*
