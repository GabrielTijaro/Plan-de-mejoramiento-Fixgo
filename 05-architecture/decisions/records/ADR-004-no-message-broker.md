# ADR-004 — Firebase Cloud Messaging Instead of a Message Broker
 
- **ID:** ADR-004
- **Date:** 2026-09-16
- **Status:** Accepted
- **Authors:** Gabriel Tijaro Jimenez
---
 
## Context
 
FixGo must notify a driver whenever their service request changes status (HU-011), and
the domain defines five events in `02-domain/domain-events.md` (`DriverRegistered`,
`VehicleRegistered`, `ServiceRequested`, `MechanicMatched`, `ServiceCompleted`). The
earlier team documentation never resolved how these are delivered: `01-context/overview.md`
and the architecture diagrams all said "FCM / Kafka" — an unresolved option presented as
if it were a decision — and a third document mentioned RabbitMQ in an example that nobody
had cleaned up. Three candidate technologies were floating in the documentation with no
decision behind any of them.
 
**Known constraints:**
- $0 infrastructure budget — a managed Kafka or RabbitMQ instance is not free
- The only consumer that genuinely needs asynchronous delivery is the mobile client
  (push notification to the driver's device)
- Cross-service calls in FixGo are few and naturally request/response (e.g. validating a
  Driver exists before creating a ServiceRequest)
- Firebase Cloud Messaging is already available through the Firebase project used for Auth
---
 
## Decision
 
**We decided:** Use **Firebase Cloud Messaging (FCM)** for push notifications to mobile
clients, and **no message broker at all** between services. Inter-service communication is
synchronous REST. Domain events are recorded and used within their owning service, not
published onto a shared bus.
 
**Justification:**
Every requirement that looked like it needed a broker turns out to be a notification to a
device, which is exactly what FCM does, at no cost, with delivery retry for offline
devices already built in (HU-011, scenario 2). Introducing Kafka or RabbitMQ would add an
infrastructure component, a delivery-guarantee problem, and an Outbox pattern to keep it
consistent — all to serve three services that could simply call each other directly.
 
---
 
## Evaluated alternatives
 
| Alternative | Pros | Cons | Reason for discarding |
|------------|------|------|-----------------------|
| FCM only, REST between services — chosen | Free; already integrated; offline retry built in; no extra infrastructure | Domain events stay local to their service, so no cross-service event history | — (chosen) |
| Apache Kafka | Durable event log; replayable; industry standard for event-driven systems | No free managed tier; operationally heavy; requires Outbox pattern for consistency | Infrastructure cost and complexity far exceed what three services with low traffic need |
| RabbitMQ | Lighter than Kafka; good routing model | Still an extra component to host and monitor; still needs Outbox | Same reason as Kafka, with less benefit |
 
---
 
## Consequences
 
**Positive:**
- Removes the unresolved "FCM / Kafka" wording from every document that carried it
- No infrastructure cost and no new operational component
- HU-011's offline-delivery acceptance criterion is satisfied by FCM natively
**Negative / Trade-offs:**
- No shared event log: if a future feature needs to react to `ServiceCompleted` from
  another service, it will need either a REST call or a revisit of this ADR
- Patterns that depend on a broker — Saga, Outbox, choreographed workflows — are
  explicitly not available, and `05-architecture/overview.md` must mark them as not adopted
**Impact on the system:**
- Affected services: `service-request` (sends notifications), all services (REST only)
- Documents that must be updated: `01-context/overview.md` (tech stack row),
  `05-architecture/overview.md` (patterns table and diagrams),
  `02-domain/domain-events.md` (consumers are in-process, not bus subscribers)
---
 
## Risks
 
| Risk | Probability | Impact | Mitigation |
|------|------------|--------|-----------|
| A future feature genuinely requires cross-service events | Medium | Medium | Supersede this ADR with a new one rather than adding a broker informally |
| Synchronous REST coupling causes cascading failures between services | Low at current scale | Medium | Revisit Circuit Breaker (currently marked not adopted) if it materializes |
 
---
 
## References
 
- `02-domain/domain-events.md` — the five domain events and their consumers
- `04-requirements/user-stories.md` — HU-011 (push notifications on status change)
- Related to: ADR-002 (architectural style), ADR-003 (data strategy)
 