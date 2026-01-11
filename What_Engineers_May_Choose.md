# Implementation Guidance: What Engineers May Choose

**Technical Flexibility Within Governance Constraints**

---

## Flexibility Map

This document clarifies where you have choices and where you do not.

---

## Storage & Database

### You May Choose

* **Relational database** (PostgreSQL, MySQL, SQL Server)
* **Document database** (MongoDB, Firestore)
* **Event sourcing** (Event store + computed state)
* **Graph database** (Neo4j)
* **Hybrid approach** (relational + document)

### You Must Guarantee

* CASE entity exists as a coherent unit
* LOCKED assets cannot be updated
* Audit log is append-only
* Timers compute correctly
* Queries can find CASEs by ID, type, status, date

### You Cannot

* Scatter CASE data across multiple systems with no single source of truth
* Store assets in mutable blobs with no version tracking
* Delete audit entries
* Reorder audit entries after creation

---

## Timer Implementation

### You May Choose

* **Scheduled job** that checks deadlines every hour
* **Event-driven** triggers when deadline passes
* **Lazy evaluation** (check deadline when CASE is accessed)
* **Persistent timers** (stored computed deadlines)

### You Must Guarantee

* Deadlines are computed correctly per business day rules
* Enforcement happens (deadline trigger = consequence)
* All computations are auditable
* Holiday and closure calendars are configurable per municipality

### You Cannot

* Skip deadline checks "for performance"
* Allow bypass of deadline consequences
* Have deadline enforcement be optional or configurable away

---

## State Machine Implementation

### You May Choose

* **Finite state machine** (explicit states and transitions)
* **Graph-based** (nodes and edges)
* **Event sourcing** (derive state from event log)
* **Hybrid** (persisted state + event log for audit)

### You Must Guarantee

* Only permitted transitions are allowed
* All transitions are logged
* Gates are enforced before transition
* State is readable (current state is queryable)

### You Cannot

* Allow transition without gate evaluation
* Skip audit logging of transitions
* Allow backwards transitions (e.g., from CLOSED to OPEN in v1.0)

---

## Asset Storage

### You May Choose

* **File system** (with version control)
* **Blob storage** (S3, Azure Blob, etc.)
* **Database** (as BLOB or text field)
* **Document store** (embedded in CASE record)
* **Content management system** (with versioning)

### You Must Guarantee

* Assets are linked to CaseID
* Asset versions are maintained (version chain)
* LOCKED assets cannot be modified
* Retention class determines storage lifetime
* Assets are retrievable by AssetID

### You Cannot

* Store assets outside CASE boundaries
* Lose version history
* Allow in-place editing of LOCKED assets
* Mix different retention classes in same storage location without clear markers

---

## API Design

### You May Choose

* **REST API** (HTTP with JSON)
* **GraphQL**
* **gRPC**
* **Event streaming** (Kafka, pub-sub)
* **Synchronous or asynchronous**

### You Must Guarantee

* CASEs can be created, read, transitioned
* Assets can be created, locked, versioned
* Audit entries can be queried
* Timers can be read
* No API endpoint allows asset mutation after lock
* No API endpoint allows audit entry deletion

### You Cannot

* Have API endpoints that bypass gates
* Allow direct manipulation of critical fields (CaseID, ClosureReason) without proper authority
* Expose version chains in confusing ways
* Require special tools to verify audit completeness

---

## Integration Patterns

### You May Choose

* **Batch ETL** — nightly pull from external systems
* **Real-time webhook** — external system notifies you
* **Polling** — regular check of external system
* **Hybrid** — real-time for critical, batch for non-critical

### You Must Guarantee

* Integration points are logged
* External data is imported into VAULT structures (CASE, assets, audit)
* You own the data after import (VAULT is source of truth)
* Integration errors are auditable

### You Cannot

* Keep critical decision logic in external system
* Have VAULT serve as "pass-through" to external system
* Lose audit trail during integration

---

## User Interface

### You May Choose

* **Web application** (React, Vue, Angular)
* **Native mobile** (iOS, Android)
* **Desktop** (Electron, WPF)
* **Command-line interface**
* **Custom UX** for different user roles (staff, public, auditor)

### You Must Guarantee

* UI displays accurate CASE state
* UI prevents invalid transitions (gates are enforced)
* UI does not allow editing of LOCKED assets
* UI clearly indicates retention class and immutability status
* UI audit trail is queryable (not just admin view)

### You Cannot

* Build UI that allows bypassing gates
* Hide asset lock status
* Simplify audit trail to the point of uselessness
* Make CASE data confusing (structure should be clear in UI)

---

## Performance Optimization

### You May Choose

* **Caching** — cache CASE state, audit queries, etc.
* **Indexing** — optimize common queries
* **Partitioning** — split large datasets
* **Denormalization** — redundant copies for performance
* **Archival** — move old CASEs to cold storage

### You Must Guarantee

* Cache invalidation is correct (no stale data)
* Indexes maintain consistency
* Denormalized copies are kept in sync
* Archived data is still queryable if needed
* Performance optimization does not weaken governance

### You Cannot

* Skip audit logging for performance
* Cache LOCKED asset content without detecting changes to source
* Optimize away gate evaluations
* Claim "performance" as reason to bypass deadline checks

---

## Security Implementation

### You May Choose

* **Role-based access control** (RBAC)
* **Attribute-based access control** (ABAC)
* **OAuth 2.0** — external authentication
* **SAML** — enterprise authentication
* **Encryption at rest**
* **Encryption in transit**
* **Audit log signing** (cryptographic verification)

### You Must Guarantee

* Only authorized actors can perform actions
* Actor identity is logged accurately
* Sensitive data (PII, exempt info) is handled per policy
* Audit log integrity can be verified (if using signatures)

### You Cannot

* Allow anonymous modifications (all actions must have actor)
* Weak password policies for admin access
* Unencrypted storage of sensitive data
* Skip encryption in transit

---

## Operations and Monitoring

### You May Choose

* **Alerts** — email, Slack, PagerDuty, etc.
* **Dashboards** — Grafana, CloudWatch, custom
* **Log aggregation** — ELK, Splunk, CloudWatch
* **Health checks** — endpoint monitoring
* **Automated recovery** — self-healing failures

### You Must Guarantee

* Critical errors are alertable (deadlines missed, gate failures, audit issues)
* System health is monitorable
* Audit completeness can be verified
* Errors are recorded and recoverable

### You Cannot

* Silently fail deadline enforcement
* Have gates fail without alerting
* Lose audit entries due to operational failure

---

## Testing and Validation

### You Should Verify

* **Business day calendar** — test with holidays, closures
* **Deadline computation** — test T10, T20, T25 with real dates
* **State transitions** — test all valid and invalid paths
* **Asset locking** — verify LOCKED assets cannot be edited
* **Audit completeness** — verify every action is logged
* **Gate enforcement** — verify gates block invalid transitions
* **Retention policies** — verify correct-class assets per module

### You Should NOT Skip

* Testing deadline drift (off-by-one errors are common)
* Testing edge cases (weekends, holidays, midnight boundaries)
* Testing multi-step processes (does state machine handle long cases?)
* Testing audit integrity (can audit log be corrupted?)

---

## Summary

**Governance (Fixed):**
* CASE structure
* Timer enforcement
* State transitions and gates
* Asset locking
* Audit completeness
* Retention classes

**Implementation (Flexible):**
* How you store data
* How you compute timers (as long as correct)
* How you implement state machine (as long as gates work)
* How you expose data (API, UI, reports)
* How you integrate with other systems
* How you monitor and alert

**Test against governance, not against implementation details.**

---

*If you are unsure whether something is flexible or fixed, ask. Better to escalate early than discover at code review that you've implemented something the wrong way.*
