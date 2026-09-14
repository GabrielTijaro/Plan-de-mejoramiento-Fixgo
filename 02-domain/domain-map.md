# Domain Map — Bounded Contexts

> **What to fill in here:** The domain map is the central DDD (Domain-Driven Design) artifact.
> It defines the system's boundaries and how they relate to each other.
> Build it first with the team and domain experts in an Event Storming session.

## Before filling in this document: Event Storming

**Event Storming** is a collaborative workshop for modeling the domain before writing code.
It lasts 2–4 hours with the whole team (dev + PO + business expert).

**Materials:** Long wall, 4-color sticky notes, markers.

**Standard colors:**
| Color | Represents | Example |
|-------|-----------|---------|
| 🟠 Orange | **Domain events** (something that happened, past tense) | `AppointmentScheduled`, `PaymentReceived` |
| 🔵 Blue | **Commands** (action that triggers the event) | `ScheduleAppointment`, `ProcessPayment` |
| 🟡 Yellow | **Actors** (who executes the command) | `Patient`, `Doctor`, `Admin` |
| 🩷 Pink | **External systems** or integration points | `Payment Gateway`, `Email SMTP` |

**Session steps:**
1. (30 min) Post all events that occur in the business, in chronological order, on the wall
2. (30 min) Identify which command or actor triggers each event
3. (45 min) Group related events — each group is a candidate Bounded Context
4. (30 min) Draw relationships between Bounded Contexts (who depends on whom)
5. (30 min) Discuss the resulting map and agree on names

**Result:** The session output directly feeds the 3 documents in `02-domain/`:
- Identified events → `domain-events.md`
- Entities and their rules → `entities-and-rules.md`
- Bounded Contexts and their map → this document

---

---

## 1. Domain overview

> One paragraph of context about the business and what problem the system solves.
> Write it without technical terms — it must be readable by a business expert.

```
[Describe the business domain here. E.g.: "The system manages the complete cycle of
[X] reservations, from the customer's request through to confirmation and billing."]
```

---

## 2. Identified Bounded Contexts

A **Bounded Context** is the explicit boundary within which a particular domain model
has consistent meaning. Each bounded context has its own Ubiquitous Language.

> **Signs of a good bounded context:**
> - Has a clear responsible team
> - Has its own database
> - Can be deployed independently
> - The same term in two different contexts can mean different things

### Bounded Context: User Management 

| Field | Value |
|-------|-------|
| **Name** | User Management |
| **Responsibility** | User/mechanic registration, authentication, roles, and vehicle records |
| **Owning team** | Individual remediation (this apprentice) |
| **Microservice(s)** | `auth-service` |
| **Database** | MySQL |
| **Ubiquitous Language** | Driver, Mechanic, Vehicle, Role, Verification |

**Context-specific terms (Ubiquitous Language):**

| Term | Meaning in THIS context | Different in another context? |
|------|------------------------|-------------------------------|
| Verification | Admin approval that unlocks a Mechanic to receive requests | No |

### Bounded Context: Matchmaking & Dispatch

| Field | Value |
|-------|-------|
| **Name** | Matchmaking & Dispatch |
| **Responsibility** | Service request lifecycle, matching to a nearby mechanic, and live location sync |
| **Owning team** | Individual remediation (this apprentice) |
| **Microservice(s)** | `service-request` |
| **Database** | MySQL (request data) + Firebase Realtime Database (live GPS, ephemeral) |
| **Ubiquitous Language** | ServiceRequest, GPSLocation, MatchResult |

### Bounded Context: Service Execution

| Field | Value |
|-------|-------|
| **Name** | Service Execution |
| **Responsibility** | On-site diagnostic and closure of a service request |
| **Owning team** | Individual remediation (this apprentice) |
| **Microservice(s)** | `service-execution` |
| **Database** | MySQL |
| **Ubiquitous Language** | Diagnostic, CompletionRecord |

---
---

## 3. Context Map

The Context Map shows relationships between bounded contexts. Relationships define
how contexts communicate and who holds the "power" in the integration.

```
┌──────────────────────┐        ┌─────────────────────────────────┐
│  User Management     │        │  Matchmaking & Dispatch         │
│  (auth-service)       │──────▶│  (service-request)              │
│                       │  U→D  │                                 │
│  Driver/Mechanic/     │        │  ServiceRequested →            │
│  Vehicle identity     │        │  MechanicMatched               │
└──────────────────────┘        └───────────────┬─────────────────┘
                                                 │ U→D
                                                 ▼
                                 ┌───────────────────────────────┐
                                 │  Service Execution             │
                                 │  (service-execution)           │
                                 │                                │
                                 │  Diagnostic & closure          │
                                 └───────────────────────────────┘
```

### Context relationship types

|| Type | Symbol | Description | Used in FixGo? |
|------|--------|-------------|---------|
| **Upstream → Downstream** | `U → D` | Upstream provides, Downstream consumes and depends on it | Yes — both relationships below |
| **Shared Kernel** | `SK` | Two contexts share part of the model | No |
| **Customer/Supplier** | `C/S` | Supplier negotiates with Customer on the contract | No |
| **Conformist** | `CONF` | Downstream adopts Upstream's model without negotiating | No |
| **Anti-Corruption Layer** | `ACL` | Downstream translates Upstream's model to protect itself | No |
| **Open Host Service** | `OHS` | Upstream publishes a stable, published protocol | No — internal REST/events only, not a public API |

### Relationships table

| Context A | Relationship | Context B | Communication channel | Contract |
|-----------|-------------|-----------|----------------------|---------|
| User Management | U → D | Matchmaking & Dispatch | REST | Internal API |
| Matchmaking & Dispatch | U → D | Service Execution | Event | `ServiceRequested` / `MechanicMatched` |
 

---

## 4. Core Domain, Supporting, Generic

DDD classifies subdomains by their strategic value:

| Bounded Context | Type | Justification |
|----------------|------|---------------|
| Matchmaking & Dispatch | Core | Real-time matching is FixGo's main value proposition |
| Service Execution | Supporting | Completes the flow but is not the differentiator |
| User Management | Generic | Standard authentication/profile management, delegated to Firebase |


---

## 5. Modeling decisions

### How were these decisions made?

No formal Event Storming workshop was held — the bounded contexts were derived directly
from the 7 SRS modules and validated individually against the roles and entities each
module describes.

### Key decisions and discarded alternatives

| Decision | Discarded alternative | Reason |
|----------|----------------------|--------|
| Merge Vehicle management into User Management instead of a separate context | A standalone "Vehicle Management" context | A vehicle has no independent lifecycle from its Driver's account |
| Single `service-request` microservice for matching + live GPS | Separate `geolocation-service` | The data model has no independent GPS schema — it's ephemeral data inside the same service |

### Key decisions and discarded alternatives

| Decision | Discarded alternative | Reason |
|----------|----------------------|--------|
| [Separate Context A and B] | [Have them in one] | [Business logic is different and they evolve at different rates] |

---

## 6. How to update this map

1. Before adding a new microservice, verify whether it belongs to an existing bounded context.
2. If a context's ubiquitous language is changing, review whether the context should be split.
3. Run an Event Storming session every time the domain changes significantly.
4. The context map MUST be synchronized with the C4 system-level diagram (`05-architecture/overview.md`).

> **Important correlation:** The bounded contexts in this document →
> Microservices in `09-microservices/service-catalog.md` →
> C4 diagrams in `08-uml/` →
> Service separation ADRs in `05-architecture/decisions/`

## Correlations
 
- Entities and rules → `02-domain/entities-and-rules.md`
- Domain events → `02-domain/domain-events.md`
- Data ownership matrix → `06-data/models.md`  
