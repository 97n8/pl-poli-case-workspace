# Technology Trade-Off Analysis

**Informed Choices for VAULT Implementation**

---

## Overview

You have flexibility in technology choices. This document shows trade-offs for each choice.

When choosing, understand what you gain and what you lose.

---

## Database Choice

### Option 1: Relational (PostgreSQL, MySQL)

**What You Get:**
* ACID guarantees (if deadline write fails, entire transition rolls back)
* Strong schema validation (CaseID cannot be null)
* Complex queries (find all CASEs by status + date range in one query)
* Mature tooling (backups, monitoring, replication)
* Good audit trail support (temporal tables, audit log tables)

**What You Lose:**
* Flexibility (schema changes require migration)
* Horizontal scaling (sharding is complex)
* Document flexibility (must define all fields upfront)

**Best For:** PRR, FISCAL, TIME (structured, high consistency)

**Trade-off Rating:** ⭐⭐⭐⭐⭐ (excellent for VAULT)

---

### Option 2: Document Database (MongoDB, Firestore)

**What You Get:**
* Schema flexibility (add fields without migration)
* Horizontal scaling (sharding is built-in)
* Fast writes (optimized for high throughput)
* Nested data (easy to store CASE + assets together)

**What You Lose:**
* ACID guarantees on complex operations (workarounds needed)
* Complex queries (may require application-level filtering)
* Audit trail (no built-in versioning; must implement)
* Consistency (eventual consistency models require care)

**Gotchas for VAULT:**
* Deadline enforcement requires strong consistency (cannot be eventual)
* Audit log immutability is harder (you must prevent updates)
* LOCKED asset immutability requires application logic (not enforced by DB)

**Best For:** High-volume, low-consistency workloads (not PRR/FISCAL/TIME)

**Trade-off Rating:** ⭐⭐⭐ (possible but requires extra work to guarantee governance)

---

### Option 3: Event Sourcing (Event Store)

**What You Get:**
* Immutable audit trail by design (everything is an event)
* Perfect temporal queries ("what was CASE state on Feb 7?")
* Replay capability (rebuild state from events)
* Strong consistency (events are append-only facts)

**What You Lose:**
* Complexity (computing state requires event replay)
* Storage (events take more space than final state)
* Query performance (may need CQRS pattern for queries)
* Operational burden (event stream infrastructure)

**Gotchas for VAULT:**
* Event sourcing is overkill for deadline enforcement (doesn't add value)
* Requires CQRS (command query responsibility segregation) to handle queries
* Operational complexity may exceed benefit

**Best For:** High-compliance, audit-heavy workloads (appeals, amendments, rewrites)

**Trade-off Rating:** ⭐⭐⭐⭐ (excellent for governance, complex operationally)

---

### Recommendation

**Tier 1 (Likely Choice):** PostgreSQL or MySQL
* Proven, stable, well-understood
* Meets all VAULT governance requirements
* Operational burden is low
* Best bang for buck

**Tier 2 (Advanced Choice):** Event sourcing with CQRS
* If you need deep audit trails and temporal queries
* If you plan multi-year data retention with frequent audits
* Accept operational complexity

**Avoid:** Pure MongoDB without ACID or strong consistency layer
* Document stores are flexible but governance requires trade-offs

---

## Timer Implementation

### Option 1: Scheduled Job (Batch)

**What You Get:**
* Simple to implement (run every hour, check all open CASEs)
* Cheap (runs when idle)
* Easy to understand and debug
* No event infrastructure needed

**What You Lose:**
* Latency (deadline miss not detected until next job run)
* Resource spike when job runs
* Missed deadlines not immediately enforced (lag time)

**Example:**
```
Every hour at :00:
  1. Find all open CASEs
  2. For each: check T10, T20, T25, T90 timers
  3. If missed: execute consequence
```

**Gotchas:**
* If job fails, deadlines are not enforced until next run
* Deadline enforcement can lag by up to 59 minutes

**Best For:** Low-volume deployments (< 1000 open CASEs)

**Trade-off Rating:** ⭐⭐⭐⭐ (simple, acceptable for v1.0)

---

### Option 2: Event-Driven (Real-Time)

**What You Get:**
* Immediate enforcement (deadline is checked when state changes)
* No background job (cheaper, simpler operationally)
* Precise (enforcement happens right at deadline moment)
* Reactive (only processes CASEs that changed)

**What You Lose:**
* Complexity (must integrate deadline checks into every state transition)
* Dependency on external event system (if event queue is down, deadlines might not be checked)

**Example:**
```
When transition attempted:
  1. Check T10, T20, T25, T90
  2. If any deadline missed: enforce consequence
  3. Emit event "deadline enforced"
  4. Allow/deny transition
```

**Gotchas:**
* If operator doesn't trigger a transition, deadline is not checked
* Requires message queue reliability

**Best For:** High-volume deployments (> 10K open CASEs)

**Trade-off Rating:** ⭐⭐⭐⭐ (more complex, more responsive)

---

### Option 3: Lazy Evaluation

**What You Get:**
* Zero operational overhead (no background jobs)
* No infrastructure needed
* Deadlines checked on-demand

**What You Lose:**
* Deadlines only checked when CASE is accessed
* If no one touches a CASE for 30 days, deadline miss is not detected
* Enforcement is delayed unpredictably

**Example:**
```
When CASE is accessed:
  1. Check T10, T20, T25, T90
  2. If any deadline missed: enforce consequence
  3. Return CASE
```

**Gotchas:**
* Violates governance (deadline consequence should trigger automatically, not on-demand)
* Operator can avoid deadline enforcement by not opening CASE

**Best For:** Not recommended for VAULT

**Trade-off Rating:** ⭐⭐ (too risky; violates enforcement principle)

---

### Recommendation

**For v1.0:** Scheduled job (hourly)
* Simple to implement
* Acceptable enforcement latency
* Scales to 10K+ CASEs

**For future:** Event-driven
* If latency becomes issue
* Real-time enforcement is better governance
* Complexity is justified at scale

**Avoid:** Lazy evaluation (governance requires active enforcement)

---

## Asset Storage

### Option 1: Database Blob

**What You Get:**
* Everything in one place (CASE + assets)
* ACID guarantees (asset and CASE state are atomic)
* Easy backup (one DB backup covers everything)
* Simple querying (assets are in same table as CASE)

**What You Lose:**
* Scalability (database can become huge)
* Performance (large blobs slow down queries)
* Flexibility (all assets share same schema)

**Best For:** Small to medium deployments (< 10M assets)

**Trade-off Rating:** ⭐⭐⭐⭐ (simple, works well)

---

### Option 2: File System (with Versioning)

**What You Get:**
* Scalability (filesystem can handle large files)
* Flexibility (any file type)
* Performance (fast for binary assets like PDFs)
* Separation of concerns (data separate from metadata)

**What You Lose:**
* ACID consistency (file write and DB write may diverge)
* Backup complexity (must back up both DB and files)
* Querying (must read file to search content)

**Gotchas:**
* If file write succeeds but DB write fails, you have orphaned files
* Immutability requires application logic (filesystem permissions not enough)

**Best For:** High-volume document storage (> 10M large assets)

**Trade-off Rating:** ⭐⭐⭐⭐ (scalable, requires care)

---

### Option 3: Object Storage (S3, Azure Blob)

**What You Get:**
* Unlimited scalability
* Built-in versioning (good for LOCKED asset immutability)
* Good performance for large files
* Mature disaster recovery

**What You Lose:**
* Latency (cloud storage is slower than local)
* Cost (pay per request, per GB)
* Dependency on external service
* Consistency guarantees (eventual consistency)

**Gotchas:**
* Object versioning does not prevent deletion (requires bucket policies)
* Costs add up with high request volume

**Best For:** Very large deployments (100M+ assets) or cloud-native

**Trade-off Rating:** ⭐⭐⭐⭐ (excellent for scale, higher cost)

---

### Recommendation

**Tier 1 (Common):** Database blob
* Most VAULT deployments start here
* Simple, proven, sufficient

**Tier 2 (Scale):** File system or S3
* When individual assets are large (PDFs, videos)
* When volume exceeds 10M assets
* When backup/restore time becomes issue

---

## Immutability Enforcement

### Option 1: Application Logic (Checks Before Update)

**What You Get:**
* Works with any database
* Simple to understand
* Flexible (can have exceptions)

**What You Lose:**
* Relies on application (if bug, immutability is broken)
* No database-level protection

**Example:**
```python
if asset.locked:
  raise PermissionError("Cannot update locked asset")
update_asset(asset, new_content)
```

**Gotchas:**
* A direct database UPDATE bypasses this check
* Multiple applications must all implement same check

**Trade-off Rating:** ⭐⭐⭐ (works, but trust issue)

---

### Option 2: Database Constraint (Stored Procedure or Trigger)

**What You Get:**
* Database enforces immutability (cannot be bypassed)
* Works with any application
* Proven technology (triggers are standard)

**What You Lose:**
* Complexity (database-level code)
* Debugging (trigger logic can be hard to trace)

**Example:**
```sql
CREATE TRIGGER prevent_locked_update
ON assets
BEFORE UPDATE
AS
  IF EXISTS (SELECT 1 FROM inserted WHERE locked = 1)
    ROLLBACK
    THROW 50001, 'Cannot update locked asset', 1
```

**Trade-off Rating:** ⭐⭐⭐⭐ (safe, proven)

---

### Option 3: Object Storage Lock (If Using S3/Azure)

**What You Get:**
* Object-level immutability
* No code required
* Cannot be overridden

**What You Lose:**
* Specific to cloud storage
* Cannot be unlocked (permanent)

**Example:**
```
Asset stored in S3
Enable: "Object Lock" → "Legal Hold"
Result: Cannot be deleted or modified
```

**Trade-off Rating:** ⭐⭐⭐⭐⭐ (excellent if using cloud storage)

---

### Recommendation

Use **database constraint** for relational databases.

Use **object lock** if using S3/Azure.

Always have **application-level check** as secondary protection.

Never rely on **application logic alone**.

---

## API Design

### Option 1: REST (HTTP JSON)

**What You Get:**
* Universal (every language has HTTP client)
* Stateless (easy to scale)
* Cacheable (HTTP caching works)
* Simple (humans can test with curl)

**What You Lose:**
* Verbosity (multiple calls needed for related data)
* Not ideal for complex queries

**Best For:** Most deployments

**Trade-off Rating:** ⭐⭐⭐⭐⭐ (proven, ubiquitous)

---

### Option 2: GraphQL

**What You Get:**
* Flexible queries (client asks for what it needs)
* Single request (get CASE + assets + audit in one call)
* Strongly typed (schema validation)

**What You Lose:**
* Complexity (GraphQL is powerful but steep learning curve)
* Query risk (complex queries can be expensive)

**Best For:** Complex client applications (UI, dashboards)

**Trade-off Rating:** ⭐⭐⭐⭐ (powerful, not necessary for v1.0)

---

### Option 3: gRPC

**What You Get:**
* High performance (binary protocol)
* Streaming support (real-time updates)
* Strongly typed (protobuf)

**What You Lose:**
* Complexity (requires code generation)
* Debugging difficulty (binary protocol)
* Not browser-friendly

**Best For:** High-performance internal APIs (service-to-service)

**Trade-off Rating:** ⭐⭐⭐ (excellent for performance, overkill for v1.0)

---

### Recommendation

**Tier 1:** REST (HTTP JSON)
* Start here; most teams know it
* Sufficient for v1.0

**Tier 2:** GraphQL
* If UI needs complex queries
* Do not add for backend integration

**Avoid:** gRPC for public API (use for internal services only)

---

## Escalation SLA Tiers

### Tier 1: Critical Blocker (< 24 hours)

**Examples:**
* Audit log is down
* LOCKED asset was mutated
* Deadline computation is wrong

**Escalation:** Page on-call
**Response:** < 2 hours
**Resolution:** < 24 hours

---

### Tier 2: High Priority (< 3 business days)

**Examples:**
* Governance gap (cannot proceed)
* Persistent database error
* Encoding introduces new risk

**Escalation:** Email to PublicLogic
**Response:** Next business day
**Resolution:** < 3 business days

---

### Tier 3: Normal (< 1 week)

**Examples:**
* Enhancement request
* Minor clarification
* Configuration question

**Escalation:** Email to PublicLogic
**Response:** < 3 business days
**Resolution:** < 1 week

---

## Summary: Technology Trade-Offs

| Choice | Gain | Lose | Rating |
|--------|------|------|--------|
| **PostgreSQL** | ACID, proven | Scalability | ⭐⭐⭐⭐⭐ |
| **MongoDB** | Flexibility, scale | Consistency | ⭐⭐⭐ |
| **Event Sourcing** | Audit, temporal | Complexity | ⭐⭐⭐⭐ |
| **Scheduled Deadline** | Simple | Latency | ⭐⭐⭐⭐ |
| **Event-Driven** | Real-time | Complex | ⭐⭐⭐⭐ |
| **DB Blob Assets** | Simple | Scale | ⭐⭐⭐⭐ |
| **File System** | Scalable | Consistency | ⭐⭐⭐⭐ |
| **S3 Storage** | Unlimited scale | Cost | ⭐⭐⭐⭐ |
| **DB Constraint** | Safe | Complexity | ⭐⭐⭐⭐ |
| **REST API** | Universal | Verbosity | ⭐⭐⭐⭐⭐ |
| **GraphQL** | Flexible | Complexity | ⭐⭐⭐⭐ |

---

## Recommended v1.0 Stack

**Proven, simple, sufficient:**

* Database: PostgreSQL
* Timer computation: Scheduled job (hourly)
* Asset storage: Database blob
* Immutability: Database constraint + application check
* API: REST (JSON)
* Error handling: Audit everything; alert on critical

**This stack is:**
* Easy to implement
* Easy to debug
* Easy to operate
* Sufficient for 10K+ CASEs
* Can scale to more complex setup later

---

*Technology choices are not governance. Choose what works for your team. But understand the trade-offs.*
