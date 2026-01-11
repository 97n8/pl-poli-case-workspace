# Implementation Guidance: What Is Non-Negotiable

**Rules That Cannot Be Bent, Bypassed, or Negotiated Away**

---

## Non-Negotiable Rules

### 1. CASE Is the Root Container

**Rule:** Every work unit lives inside exactly one CASE. No work exists outside CASE boundaries.

**You Cannot:**
* Create workflows that do not create CASEs
* Scatter work across multiple systems without a CASE binding them
* Have ad-hoc processes that skip CASE creation

**Example (Violation):**
"We'll handle this PRR through email without creating a CASE."
❌ Wrong. Every PRR gets a CASE.

---

### 2. Timers Are Enforced

**Rule:** Statutory deadlines trigger consequences automatically. No discretion.

**You Cannot:**
* Skip deadline enforcement "because it's not critical"
* Allow operators to override deadline consequences
* Have deadline enforcement be optional or configurable
* Apply deadlines as "reminders" instead of enforcements

**Example (Violation):**
"Let's not waive fees even though T10 was missed, because the RAO was busy."
❌ Wrong. T10 miss triggers fee waiver automatically.

---

### 3. LOCKED Assets Cannot Be Edited

**Rule:** Once an asset is LOCKED, its content cannot be changed. Period.

**You Cannot:**
* Allow in-place editing of LOCKED assets
* "Unlock" assets to fix errors
* Silently overwrite LOCKED content
* Use version control that allows rewriting history of LOCKED assets

**How to correct errors:** Create a new version. Link as version chain. Lock both.

---

### 4. Audit Log Is Immutable

**Rule:** Audit log entries cannot be edited, deleted, or reordered after creation.

**You Cannot:**
* Update an audit entry to change timestamps or actor
* Delete audit entries
* Reorder audit log
* Allow anyone to edit historical audit entries

**Implementation:** Append-only storage. Period.

---

### 5. CASE Cannot Reopen (v1.0)

**Rule:** Once a CASE is closed, it cannot be reopened. New work = new CASE.

**You Cannot:**
* Allow closed CASEs to transition back to OPEN
* Resume work on closed CASEs
* Merge related CASEs

**Exception:** v1.1 will handle appeals. Until then, appeals are manual escalations outside VAULT.

---

### 6. Gates Must Be Enforced Before Transition

**Rule:** Before state transition, all transition blockers must be empty.

**You Cannot:**
* Allow transition with blockers present
* Skip gate evaluation
* Have gates be suggestions
* Bypass gates "just this once"

**Example (Gate):**
"If T25 forecast shows completion > 25 days, AND no extension agreement, THEN block transition to GATHERING."

This gate is non-negotiable.

---

### 7. Record Classification Must Be Accurate

**Rule:** Every asset must be classified as KEEPER, REFERENCE, or TRANSACTIONAL. Classification determines retention and mutation rules.

**You Cannot:**
* Misclassify records (call KEEPER "REFERENCE" to delete sooner)
* Allow reclassification to avoid retention rules
* Store different retention classes together without clear markers

---

### 8. Statutory Authority Is Binding

**Rule:** If statute requires something, VAULT enforces it. You do not have the option to skip it.

**You Cannot:**
* Reinterpret statute to weaken enforcement
* Skip statutory requirements for "efficiency"
* Claim "business necessity" overrides statute

**Example (Statute):**
M.G.L. c. 66, §10 says initial response due in 10 business days. This is not negotiable. You cannot say "Our town will aim for 15 days."

---

### 9. Actor Identity Must Be Logged

**Rule:** Every action has an actor. No anonymous changes.

**You Cannot:**
* Log actions without identifying who did them
* Use generic "system" account for human decisions
* Allow actions to be unattributed

---

### 10. Governance Gap Escalation Is Mandatory

**Rule:** If encoding introduces ambiguity or risk, you must escalate. You cannot interpret around gaps.

**You Cannot:**
* Make up answers to governance questions
* Guess at statutory interpretation
* Deploy with ambiguity unresolved

**Process:**
1. Identify ambiguity
2. Document it
3. Send to info@publiclogic.com
4. Wait for clarification
5. Resume with clarification in hand

---

## Examples of Violations

### Violation 1: Bypassing T10 Deadline

**Scenario:** PRR received Tuesday. T10 due date is Tuesday of following week. It's now Thursday before due date. Response is not ready.

**Temptation:** "Let's just not charge fees for this one, even though we miss the deadline, because we're close."

**Reality:** You cannot. If T10 is missed, fees are waived. Full stop. If you are close, work faster. If you will miss, plan accordingly (e.g., request extension).

**Correct approach:** Understand deadline weeks in advance. Plan accordingly.

---

### Violation 2: Unlocking an Asset to Fix an Error

**Scenario:** Withholding letter was locked, but we found an error. We want to fix the letter in place.

**Temptation:** "Let's unlock the asset, fix the text, and lock it again. Nobody will know."

**Reality:** You cannot. LOCKED means immutable. Create version 2. Link to version 1. Lock both. Audit trail shows both.

**Correct approach:** Accept that fixing errors creates new versions. This is the cost of immutability.

---

### Violation 3: Deleting Old Audit Entries

**Scenario:** Audit log has grown large. We want to clean up old entries to improve performance.

**Temptation:** "Let's archive and delete entries older than 2 years."

**Reality:** You cannot. Audit log is permanent. Move old entries to cold storage if needed, but do not delete.

**Correct approach:** Retention rules apply to assets, not audit log. Audit log is forever.

---

### Violation 4: Reconfiguring a Deadline

**Scenario:** PRR module sets T10 = 10 business days per statute. But our town is slow, so we want to change it to 15 days in software.

**Temptation:** "Let's add a configuration option: T10_OVERRIDE_DAYS = 15"

**Reality:** You cannot. M.G.L. c. 66, §10 specifies 10 days. You cannot change statute through software configuration. If statute changes, PublicLogic amends VAULT.

**Correct approach:** Respect statutory limits. If your town is slow, staff up earlier.

---

### Violation 5: Skipping Audit Logging for Performance

**Scenario:** Audit logging is slowing down API responses. We want to disable it during high load.

**Temptation:** "Let's batch audit entries and write them later, or skip logging transient state changes."

**Reality:** You cannot. Audit log must be complete and immediate. Every action logged before response returns.

**Correct approach:** Optimize audit logging (e.g., async writes, batching), but do not skip entries.

---

### Violation 6: Guessing at Governance Gap

**Scenario:** Spec doesn't say what to do if requester withdraws a PRR after records are gathered. We need an answer to deploy.

**Temptation:** "Let's assume we can just close the case and destroy the records."

**Reality:** You cannot guess. Escalate: "Can PRRs be withdrawn after intake? What happens to gathered records? Should we create a new closure reason?"

PublicLogic answers. You implement the answer.

**Correct approach:** Stop, escalate, wait for answer, resume.

---

## Summary

| Category | Negotiable | Non-Negotiable |
|----------|-----------|-----------------|
| How you store data | YES | — |
| Which deadlines exist | — | YES |
| How CASE is structured | — | YES |
| Whether gates work | — | YES |
| Asset locking mechanism | YES | Must prevent edits |
| Whether audit logs are complete | — | YES |
| Statute interpretation | — | YES |
| Configuration of deadlines | YES* | Statute must be honored |
| Who has access to UI | YES | Actor must be logged |

*Configuration = local holidays, cost codes, approval chains. NOT deadlines themselves.

---

*If a requirement in VAULT Core Canon or module spec appears too strict, escalate instead of working around it. Working around it creates risk.*
