
#### 1.3 Mandatory Guards

| Transition       | Guard Condition (MUST be true)                          | If False                     |
|------------------|----------------------------------------------------------|------------------------------|
| S300 → S310      | Clarification genuinely needed                           | Block                        |
| S300 → S910      | RAO attests no responsive records                        | Block                        |
| S330 → S340      | Forecast > T25 AND signed extension agreement exists     | Block                        |
| S330 → S350      | Forecast > T25 AND no agreement → petition required      | Block                        |
| S500 → S920      | All responsive records fully exempt                      | Block                        |
| S700 → S710      | Fees required AND estimate > $0                          | Block                        |
| S700 → S800      | Fees waived OR not applicable                            | Proceed only if no fees      |

---

### 2. Key Statutory Rules

- **Effective receipt date** — next business day if received on non-business day  
- **Deadlines** — T10 (initial response), T20 (petition window), T25 (max production) — all business days  
- **Tolling** — starts on clarification sent / requester delay / supervisor extension; ends on complete response; one active event only; recomputes deadlines  
- **T10 fee waiver** — if first response after T10 due date → fees permanently disabled  
- **T25 overrun** — block production unless extension agreement or Supervisor petition  
- **Timeouts** — 30 calendar days (clarification or payment) → auto-close  

---

### 3. Asset Immutability

**Locked on creation** (cannot be changed):

- ORIGINAL_REQUEST  
- NO_RECORDS_LETTER  
- WITHHOLDING_LETTER  
- RESPONSE_PACKAGE  
- EXTENSION_AGREEMENT  
- PETITION  
- PAYMENT_RECEIPT  
- DELIVERY_CONFIRMATION  

Replacement of locked assets creates new version + links via version chain.

---

### 4. Audit Log Requirements

**Must log** (at time of occurrence):

- CASE_CREATED  
- STATE_TRANSITION  
- ASSET_CREATED  
- ASSET_LOCKED  
- TIMER_COMPUTED  
- TIMER_TOLLED  
- TIMER_RESUMED  
- FEE_WAIVED  
- CASE_CLOSED  
- ERROR_LOGGED  

Log is **append-only** — no updates or deletes permitted.

---

### 5. Compliance Checklist (before production)

- [ ] All 22 states & transitions enforced  
- [ ] Guards checked at runtime  
- [ ] Dynamic MA holiday calendar used  
- [ ] T10 waiver automatic & logged  
- [ ] T25 overrun blocked without extension/petition  
- [ ] Tolling tested: start → end → recompute correct  
- [ ] Locked assets immutable  
- [ ] Audit log append-only & complete  
- [ ] Timeout scheduler runs daily  
- [ ] Error handling complete  

---

**Final Statement**  
This specification **is the contract**.  
Polimorphic **must** implement exactly these rules.  
PublicLogic will validate compliance against this document alone.

**Classification:** Confidential – PublicLogic / Polimorphic use only  
**Next Review:** Upon material law change or pilot feedback
