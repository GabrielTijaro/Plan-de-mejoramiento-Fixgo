# ADR-003 — Data Strategy: MySQL per Service, Firebase RTDB for Live GPS Only
 
- **ID:** ADR-003
- **Date:** 2026-09-16
- **Status:** Accepted
- **Authors:** Gabriel Tijaro Jimenez
---
 
## Context 
 
Each of the three services defined in ADR-002 needs to persist data, and
`service-request` additionally needs to push live GPS positions to mobile clients several
times per minute per active request. Two different documents in the team repository had
already drifted apart on this point — `06-data/models.md` said MySQL while
`09-microservices/service-catalog.md` said PostgreSQL for the same `auth-service` — and
`05-architecture/overview.md` referenced a data-strategy ADR that had never been written.
This ADR exists to close that gap and be the one place the answer lives.
 
**Known constraints:**
- $0 infrastructure budget — must fit within free tiers
- The core entities (Driver, Vehicle, Mechanic, ServiceRequest, Diagnostic) carry hard
  invariants documented in `02-domain/entities-and-rules.md` (INV-004, INV-005, INV-006,
  INV-008) that depend on referential integrity
- Live GPS coordinates are overwritten constantly and are never queried historically
- Firebase Auth is already part of the stack, so the Firebase SDK is already a dependency
---
 
## Decision
 
**We decided:** Use **MySQL** as the relational engine for all three services, each with
its own schema (Database per Service), and use **Firebase Realtime Database** exclusively
for the ephemeral live-GPS stream inside `service-request`. No other service uses Firebase
for persistence.
 
**Justification:**
The invariants in `02-domain/entities-and-rules.md` are the tiebreaker. INV-006 (unique
plate) and INV-008 (one Diagnostic per ServiceRequest) are enforced most reliably as
database constraints, not as application code that can be bypassed. Firebase RTDB is kept
to the single place where its strengths — low-latency push to mobile clients, schemaless
writes — actually matter, and where its lack of referential integrity costs nothing
because the data is transient.
 
---
 
## Evaluated alternatives
 
| Alternative | Pros | Cons | Reason for discarding |
|------------|------|------|-----------------------|
| MySQL per service + Firebase RTDB for live GPS — chosen | Free tier fits the budget; DB-level enforcement of domain invariants; Firebase SDK already present | Two data technologies to operate; GPS history not queryable for analytics | — (chosen) |
| PostgreSQL per service | Richer features (JSONB, PostGIS for geospatial queries) | No team experience with it; PostGIS is unnecessary at MVP matching scale | Adds a learning cost with no benefit at this stage |
| Firebase Firestore for everything | One technology; already integrated; no separate DB to provision | No native foreign keys; INV-004/006/008 would have to be enforced only in application code | Integrity risk too high for dispatch records that must be auditable (SRS Module 7) |
 
---
 
## Consequences
 
**Positive:**
- Resolves the MySQL/PostgreSQL contradiction between `06-data/models.md` and
  `09-microservices/service-catalog.md`
- Domain invariants are enforced by the database, not only by application logic
- Fits the $0 budget on both MySQL and Firebase free tiers
**Negative / Trade-offs:**
- Two data technologies increase operational surface area
- GPS history is discarded unless explicitly archived into MySQL later, so no
  route-replay or analytics feature is possible without adding that archival step first
**Impact on the system:**
- Affected services: `auth-service`, `service-request`, `service-execution`
- Documents that must be updated: `06-data/models.md`,
  `09-microservices/service-catalog.md` (currently still says PostgreSQL for
  `auth-service` — must be corrected to MySQL), `05-architecture/overview.md`
---
 
## Risks
 
| Risk | Probability | Impact | Mitigation |
|------|------------|--------|-----------|
| `service-catalog.md` is never corrected and the contradiction persists | Medium | Medium | This ADR is the source of truth; any document disagreeing with it is the one to fix |
| Firebase RTDB free-tier concurrent-connection limits are hit | Low at MVP scale | Medium | Revisit when concurrent active requests approach the 500 target in NFR-05 |
 
---
 
## References
 
- `02-domain/entities-and-rules.md` — INV-004, INV-005, INV-006, INV-008
- `06-data/models.md` — table-level schema per service
- `04-requirements/non-functional.md` — NFR-02 (15m GPS accuracy), NFR-05 (500 concurrent requests)
- Related to: ADR-002 (architectural style), ADR-004 (notifications without a message broker)