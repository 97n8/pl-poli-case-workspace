# VAULT Core Canon: Timer and Deadline Model

**How Deadlines Are Computed • Enforcement Without Discretion**

---

## Core Principle

**Timers are enforced, not tracked.**

This means:
* Deadlines are computed automatically from a statute or rule
* When deadline is reached, consequences execute automatically
* No human can bypass or override the consequence
* All deadline events are logged

Example: If a statute says "response due in 10 days," VAULT computes the exact date and enforces fee waiver if missed. There is no discretion.

---

## Timer Computation

### Step 1: Determine Effective Receipt Date

The deadline clock starts from **EffectiveReceiptDate**, not from actual receipt.

| Receipt Scenario | Effective Date | Why |
|------------------|----------------|-----|
| Monday, Jan 20 (business day) | Monday, Jan 20 | Same day |
| Friday, Jan 24 (business day) | Friday, Jan 24 | Same day |
| Saturday, Jan 25 | Monday, Jan 27 | Next business day |
| Sunday, Jan 26 | Monday, Jan 27 | Next business day |
| Monday, Jan 27 (holiday) | Tuesday, Jan 28 | Next business day |
| Friday, Jan 31 (before weekend) | Friday, Jan 31 | Same day (Friday counts) |

**Rule:** If received on non-business day, shift to next business day.

**Definition of business day:** Weekday (Mon-Fri) that is not a state legal holiday or municipal closure.

### Step 2: Add Business Days

Once EffectiveReceiptDate is known, add the required number of business days.

**Example:** T10 deadline with EffectiveReceiptDate = Friday, Jan 20

```
Day 1: Fri Jan 20   (receipt)
Day 2: Mon Jan 23   (weekend skip: Sat 21, Sun 22)
Day 3: Tue Jan 24
Day 4: Wed Jan 25
Day 5: Thu Jan 26
Day 6: Fri Jan 27
Day 7: Mon Jan 30   (weekend skip)
Day 8: Tue Jan 31
Day 9: Wed Feb 01
Day 10: Thu Feb 02

T10 Due Date: Thursday, Feb 02
```

**Rule:** Count only business days. Skip weekends and holidays.

### Step 3: Account for Holidays

Massachusetts legal holidays (and municipal closures) are non-business days.

```
If T10 receipt = Friday, Feb 20 (Presidents' Day observed)
Day 1: Fri Feb 20
Day 2: Mon Feb 23   (Presidents' Day Feb 21, skip Sat/Sun)
Day 3: Tue Feb 24
...
```

**Rule:** Each municipality must configure its holiday calendar and closure days.

---

## Standard VAULT Timers

### T10: Initial Response

**Definition:** Initial written response due

**Applies to:** PRR, some licensing, some other modules

**Duration:** 10 business days from EffectiveReceiptDate

**Consequence if missed:** Set `FeesAllowed = false` (permanent)

**Examples:**

| Module | What Counts as "Response" |
|--------|--------------------------|
| **PRR** | Initial letter from RAO (even if partial) |
| **CLERK** | Acknowledgment of application receipt |
| **FISCAL** | Acknowledgment of invoice |

**Enforcement rule:**
```
If CurrentDate > T10DueDate AND ResponseNotSent:
  FeesAllowed = false
  LogEvent("T10 deadline missed; fees waived")
```

---

### T20: Petition Window

**Definition:** Requester may petition Supervisor

**Applies to:** PRR (M.G.L. c. 66, §10(b))

**Duration:** 20 business days from EffectiveReceiptDate

**Consequence if missed:** Right to petition is lost

**Enforcement rule:**
```
If CurrentDate > T20DueDate AND NoPetitionFiled:
  PetitionWindowClosed = true
  LogEvent("T20 deadline passed; petition window closed")
  
If PetitionAttemptedAfterT20:
  Reject("T20 window has closed")
```

---

### T25: Municipal Production Limit

**Definition:** Municipality must produce records or justify extension

**Applies to:** PRR (M.G.L. c. 66, §10(a))

**Duration:** 25 business days from EffectiveReceiptDate (or extended period if granted)

**Consequence if missed:** Extension required

**Enforcement rule:**
```
If ForecastedCompletionDate > T25DueDate:
  IF RequesterAgreedToExtension:
    ExtensionGranted = true
    RecomputeDeadline(ExtensionEndDate)
  ELSE IF SupervisorGrantedExtension:
    ExtensionGranted = true
    RecomputeDeadline(ExtensionEndDate)
  ELSE:
    BlockTransition("Extension required; cannot proceed to delivery")
```

---

### T90: Appeal Window

**Definition:** After closure, requester may appeal to Supervisor

**Applies to:** PRR (M.G.L. c. 66, §10(d); 950 CMR 32.08(2))

**Duration:** 90 calendar days from CASE closure date

**Consequence if missed:** Right to appeal is lost

**Note:** T90 runs from closure, not from receipt (unlike T10, T20, T25).

**Enforcement rule:**
```
If AppealAttempted AND CurrentDate > (ClosureDate + 90 days):
  Reject("Appeal window has closed; T90 exceeded")
```

---

## Module-Specific Timers

Beyond the standard timers, each module may define module-specific deadlines.

**Examples:**

| Module | Timer | Duration | Consequence |
|--------|-------|----------|-------------|
| **VAULT TIME** | Payment Due | 30 days from approval | Hold paycheck |
| **VAULT FISCAL** | 3-Way Match Due | 60 days | Flag variance |
| **VAULT CLERK** | Inspection Due | 30 days | Delay approval |

Module specs define these timers and their enforcement.

---

## Tolling (Pausing the Clock)

Sometimes the deadline clock must be paused.

### When Tolling Applies

| Reason | Example | Duration |
|--------|---------|----------|
| **Clarification** | Requester must clarify scope | Until response |
| **Requester Delay** | Requester asks to hold case | Until resumption |
| **Supervisor Extension** | Supervisor grants extension | Per extension period |
| **Judicial Hold** | Litigation is pending | Per court order |

### How Tolling Works

```
Original T25 Due Date: Friday, Feb 15

Event: Clarification requested on Jan 20
Tolling Start: Jan 20
Reason: "Scope is ambiguous; need clarification"

Clarification received: Jan 25 (5 business days)
Tolling End: Jan 25
Tolled Business Days: 5

New T25 Due Date: Friday, Feb 15 + 5 = Friday, Feb 22
```

**Rule:** Tolling extends the deadline, not resets it.

### Tolling Log

Every tolling event must be logged:

```json
{
  "TollingID": "toll-2026-001",
  "CaseID": "prr-2026-001",
  "Reason": "Clarification Requested",
  "StartDate": "2026-01-20",
  "EndDate": "2026-01-25",
  "BusinessDaysTolled": 5,
  "ResumeAsset": "CLARIFICATION_RESPONSE",
  "LogEntry": "T25 tolled 5 business days per clarification protocol"
}
```

---

## Timer Computation Algorithm

Pseudocode:

```
FUNCTION ComputeDeadline(EffectiveReceiptDate, BusinessDaysRequired, Holidays, Closures)
  
  CurrentDate = EffectiveReceiptDate
  BusinessDaysElapsed = 0
  
  WHILE BusinessDaysElapsed < BusinessDaysRequired:
    CurrentDate = CurrentDate + 1 day
    
    IF IsWeekend(CurrentDate):
      CONTINUE
    ELSE IF IsHoliday(CurrentDate, Holidays):
      CONTINUE
    ELSE IF IsMunicipalClosure(CurrentDate, Closures):
      CONTINUE
    ELSE:
      BusinessDaysElapsed = BusinessDaysElapsed + 1
  
  RETURN CurrentDate  // Due date
END

FUNCTION EnforceDeadline(Timer, DueDate, CurrentDate, Action)
  
  IF CurrentDate > DueDate AND Action NOT COMPLETED:
    EXECUTE Consequence per timer definition
    LOG("Deadline missed", Timer, DueDate, Consequence)
    RETURN True  // Enforcement triggered
  ELSE:
    RETURN False  // Deadline not missed
  
END
```

---

## Enforcement Without Discretion

### Non-Negotiable Consequences

| Timer | Consequence | Discretionary? |
|-------|-------------|----------------|
| T10 miss | Fee waiver (`FeesAllowed = false`) | **NO** — automatic |
| T20 pass | Petition window closed | **NO** — automatic |
| T25 forecast | Extension required before delivery | **NO** — automatic |
| T90 pass | Appeal window closed | **NO** — automatic |

**Rule:** These consequences execute automatically. No human review or override.

### Why No Discretion?

Discretion creates risk:

* If RAO can decide "This deadline doesn't matter," deadline means nothing
* If supervisor can override fee waiver, fee protection is meaningless
* If someone can "hand-waive" an extension gate, gate is useless

Removing discretion protects staff from having to make judgment calls about statutory requirements.

---

## Drift Detection

VAULT must detect when timers are not being enforced correctly.

### Alert Conditions

| Condition | Alert |
|-----------|-------|
| T10 deadline passed; response not sent | "T10 MISS: Response overdue; fee waiver not applied" |
| T25 deadline approaching; no extension agreement | "T25 FORECAST: Completion > 25 days; extension required" |
| T20 deadline passed; petition still allowed | "T20 CLOSED: Petition window has closed" |
| Tolling event logged but not applied to deadline | "TOLLING ERROR: Tolling recorded but deadline not recomputed" |

These alerts help catch implementation errors.

---

## Configuration

Each municipality must provide:

1. **Holiday Calendar** — Which state holidays are observed
2. **Closure Days** — When office is closed (e.g., town meeting day)
3. **Business Hours** — When business day starts/ends (if relevant)

Example configuration:

```yaml
holidays:
  - "01-01"  # New Year
  - "01-20"  # MLK Day (third Monday)
  - "12-25"  # Christmas
  
closures:
  - "2026-05-06"  # Special closure
  
business_hours:
  start: "09:00"
  end: "17:00"
```

---

## Q&A

**Q: Can I request an extension beyond what statute allows?**
A: No. VAULT enforces statutory limits. Requests for extension beyond statute are escalated to Supervisor.

**Q: What if a deadline falls on a holiday?**
A: It shifts to the next business day.

**Q: Can the RAO waive a deadline?**
A: No. Deadlines are statutory. RAO cannot waive. Supervisor can extend (with authority).

**Q: If tolling is in effect, does the deadline extend?**
A: Yes. You add the tolled days back to the deadline.

**Q: What if I miss T10 but respond the next day?**
A: Too late. Fee waiver is already triggered. Response is still valid, but fees cannot be charged.

---

*Timers are how VAULT enforces statute without requiring a human to remember or decide. If you do not implement them correctly, VAULT fails.*
