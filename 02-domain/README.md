# 02 — Problem Domain

> **What is this?** The mental model of the business. It is not technology — it is understanding
> the problem the system solves before writing code. This section comes from Domain-Driven Design (DDD).

## Why this section exists

The most costly mistakes in software are not bugs — they are domain misunderstandings.
When developers do not deeply understand the business:
- Entities and invariants match real-world constraints (like the strict 3-second SLA and 15-meter GPS precision).
- Microservice boundaries are drawn correctly based on clear Bounded Contexts (`auth-service`, `service-request`, `service-execution`).
- Ubiquitous language is shared equally among developers, product owners, and business stakeholders.

This section captures domain knowledge **before** designing the architecture.

---

## Key concepts you must know

## Key concepts you must know

## Key concepts you must know

* **Entity:** Domain object with a unique identity (e.g.: a `DriverProfile` or `MechanicProfile` identified by their ID).
* **Value Object:** Object with no identity of its own, defined by its attributes (e.g.: `GPSLocation`, `Money`).
* **Aggregate:** Group of entities treated as a unit. Only the aggregate root can be referenced from outside (e.g.: `ServiceRequest`).
* **Domain Event:** Something that occurred in the business that other parts of the system must know (e.g.: `ServiceRequested`, `MechanicMatched`). They are facts, stated in past tense.
* **Bounded Context:** Area of the system where a particular model applies. Each microservice generally corresponds to a bounded context (e.g.: `service-request`).

---

## What is here and how to fill it in

### `domain-map.md` ⭐
Map of all bounded contexts and how they relate.
**Fill in:** draw the contexts as rectangles and the relationships between them
(upstream/downstream, shared kernel, anti-corruption layer).

**Format:**
```markdown
## Bounded Contexts
### Matchmaking & Dispatch
**Responsibility:** Service request lifecycle, matching to a nearby mechanic, and live location sync.
**Main entities:** ServiceRequest, MatchResult, GPSLocation.
**Owning team:** FixGo Backend Team

## Relationship map
User Management (auth-service) ──▶ Matchmaking & Dispatch (service-request) ──▶ Service Execution (service-execution)

| Context A | Relationship | Context B | Description |
|-----------|-------------|-----------|-------------|
| service-request | downstream-of | auth-service | service-request consumes identity and vehicle data to validate incoming requests. |
| service-execution | downstream-of | service-request | service-execution is triggered once a ServiceRequest is successfully assigned to a mechanic. |
```

### `entities-and-rules.md` ⭐
Catalog of entities, value objects, and business rules.
**Fill in:** for each entity: name, attributes, invariants (rules that MUST ALWAYS hold),
behaviors.

**Format:**
```markdown
## Entity: ServiceRequest
**Belongs to:** Matchmaking & Dispatch (`service-request`)
**Identifier:** requestId (UUID)

### Attributes
| Attribute | Type | Description | Required | Rules |
|-----------|------|-------------|---------|-------|
| driverId | UUID | Requester ID | Yes | Must be an active Driver |
| incidentLocation | GPSLocation | Breakdown coords | Yes | Required for dispatch algorithm |

### Business rules (invariants)
- [x] INV-002 Latency SLA: Real-time location updates and matchmaking alerts must process in under 3 seconds.

### Behaviors (domain methods)
- `acceptRequest(mechanicId)`: Validates mechanic is available, locks the request, transitions state to ACCEPTED.
```

### `domain-events.md` ⭐
List of all events that occur in the domain.
**Fill in:** event name (past tense), what triggers it, what data it carries, who consumes it.

**Format:**
```markdown
| Event | Triggered by | Data | Consumers | Bounded Context |
|-------|-------------|------|-----------|----------------|
| MechanicMatched | Algorithm assigns a mechanic within the SLA | requestId, mechanicId, eta | Service Execution, Driver UI | Matchmaking & Dispatch |
```

---

## Correlations with other sections

| This section feeds... | Why |
|-----------------------|-----|
| `05-architecture/overview.md` | Bounded contexts → microservices |
| `06-data/models.md` | Entities → data tables/collections |
| `07-api/contracts/` | Domain events → events in async APIs |
| `09-microservices/event-catalog.md` | All domain events are registered there |
| `04-requirements/user-stories.md` | Business rules → acceptance criteria |

---

## Recommended tool: Event Storming

**Event Storming** is a domain discovery workshop with sticky notes:
1. 🟠 Orange: Domain events (past tense)
2. 🔵 Blue: Commands (what triggers the event)
3. 🟡 Yellow: Actors (who executes the command)
4. 🟣 Purple: Policies (automatic reactions)
5. 🟦 Light blue: External systems

Running an Event Storming session with the team before filling in this section saves weeks of redesign.

---

## Questions this section must answer

- What are the main business entities?
- What rules can NEVER be violated in the system?
- What important events occur in the domain?
- Where are the natural boundaries of the system (for defining microservices)?
