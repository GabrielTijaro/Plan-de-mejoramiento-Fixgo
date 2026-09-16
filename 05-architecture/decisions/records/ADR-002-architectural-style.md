# ADR-002 — Microservices as the Architectural Style
 
- **ID:** ADR-002
- **Date:** 2026-09-16
- **Status:** Accepted
- **Authors:** Gabriel Tijaro Jimenez
---
 
## Context
 
FixGo's domain was decomposed into three bounded contexts in `02-domain/domain-map.md`
(User Management, Matchmaking & Dispatch, Service Execution). Before writing
`05-architecture/overview.md`, the project needs a recorded decision on whether those
contexts become independently deployable services or modules inside a single deployable
unit. Deciding this now prevents the documentation from drifting — the team repository
(`fixgo-docs`) ended up naming the same service three different ways precisely because
no ADR fixed the structure first.
 
**Known constraints:**
- $0 infrastructure budget — free tiers only
- This is an academic project with a 4-day remediation window; no production traffic yet
- The 3-second matching SLA (NFR-01) applies to one specific flow, not to the whole system
- The team's stack is Java 17 + Spring Boot, with Firebase already integrated for auth
---
 
## Decision
 
**We decided:** Structure FixGo as **three microservices behind an API Gateway** —
`auth-service`, `service-request`, and `service-execution` — each owning its own database,
communicating synchronously over REST.
 
**Justification:**
The three bounded contexts have genuinely different operational profiles.
`service-request` is the only one bound by a hard latency SLA (NFR-01) and the only one
that needs a real-time data layer; `auth-service` and `service-execution` are ordinary
CRUD workloads. Separating them means the latency-critical service can be scaled and
tuned without dragging the other two along. The tiebreaker was the Database-per-Service
requirement already asserted in `06-data/models.md`: honoring it inside a single
deployable unit would give the drawbacks of both styles at once.
 
---
 
## Evaluated alternatives
 
| Alternative | Pros | Cons | Reason for discarding |
|------------|------|------|-----------------------|
| Three microservices + API Gateway — chosen | Independent scaling of the latency-critical service; clean mapping to bounded contexts; each service owns its schema | More deployment units to operate; cross-service consistency handled manually | — (chosen) |
| Modular monolith (one deployable, three modules) | Simpler to deploy and debug; no network hops; transactions across contexts are trivial | Cannot scale `service-request` independently; conflicts with Database-per-Service already decided in `06-data/models.md` | The data-ownership decision was already made and would have to be reversed |
| Two services (merge Service Execution into Matchmaking) | Fewer moving parts; Diagnostic is tightly coupled to ServiceRequest anyway | Blurs the boundary between dispatching and on-site execution, which the SRS treats as distinct modules (3 vs 4) | Loses the domain boundary the SRS itself draws |
 
---
 
## Consequences
 
**Positive:**
- Service names are fixed once, here, and every other document must match them
- `service-request` can be optimized for the 3-second SLA without affecting the others
- Each service maps 1:1 to a bounded context, so `02-domain` stays the source of truth
**Negative / Trade-offs:**
- Cross-service operations (e.g. validating a Driver before creating a request) require
  a network call instead of a local method call
- Three deployment pipelines instead of one, once the project reaches `10-devops/`
**Impact on the system:**
- Affected services: `auth-service`, `service-request`, `service-execution`, `api-gateway`
- Documents that must be updated: `05-architecture/overview.md` (service catalog and
  diagrams), `09-microservices/service-catalog.md`, `04-requirements/traceability-matrix.md`
---
 
## Risks
 
| Risk | Probability | Impact | Mitigation |
|------|------------|--------|-----------|
| Documents drift and use different names for the same service | Medium | High | `02-domain/domain-map.md` is the single source of truth for service names; any document that disagrees is the one that is wrong |
| Network latency between services eats into the 3-second SLA | Low | High | Keep the matching path free of cross-service calls — `service-request` resolves matching against its own data |
 
---
 
## References
 
- `02-domain/domain-map.md` — the three bounded contexts this decision implements
- `04-requirements/non-functional.md` — NFR-01 (3s matching SLA), NFR-05 (500 concurrent requests)
- Related to: ADR-003 (data strategy), ADR-004 (notifications without a message broker)
   