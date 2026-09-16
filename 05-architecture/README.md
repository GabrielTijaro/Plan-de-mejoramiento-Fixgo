# 05 — Architecture

> **What is this?** The system's design decisions: how it is organized, why,
> what alternatives were evaluated, and how it is deployed. ADRs are the treasure of this section.

## Why this section exists

A system's architecture is the set of decisions that are hard to change later.
Documenting them has three benefits:
1. **New team members** understand the system without having to ask everything from scratch
2. **The team** does not repeat already-resolved discussions
3. **Years later**, everyone remembers why each decision was made

---

## What is here and how to fill it in

### `overview.md` ⭐ (Start here)
High-level view of the complete system.
**Fill in:** C4 Level 1 (System) and Level 2 (Container) diagram, list of microservices with
each one's responsibility, how they communicate (sync/async), technologies per layer.

**Recommended format:**
```markdown
## Architecture diagram
```text
                  ┌──────────────────────┐
                  │   Android App        │
                  │ (Driver / Mechanic)  │
                  └──────────┬───────────┘
                             │
                  ┌──────────▼───────────┐
                  │     API Gateway      │
                  │        :8080         │
                  └──────────┬───────────┘
        ┌────────────────────┼────────────────────┐
        │                    │                    │
┌───────▼──────┐   ┌─────────▼────────┐  ┌────────▼──────────┐
│ auth-service │   │ service-request  │  │ service-execution │
│    :3001     │   │      :3002       │  │      :3003        │
└───────┬──────┘   └─────────┬────────┘  └────────┬──────────┘
        │                    │                    │
   ┌────▼────┐      ┌────────▼────────┐      ┌────▼────┐
   │  MySQL  │      │ MySQL + Firebase│      │  MySQL  │
   └─────────┘      │  Realtime DB    │      └─────────┘
                    └─────────────────┘
```
 
**Service catalog (resolved — see `overview.md`):**
 
| Service | What it does | Stack | Engine |
|---|---|---|---|
| `api-gateway` | Routing, JWT validation, rate limiting | Spring Cloud Gateway | — |
| `auth-service` | Registration, login, roles, Driver/Mechanic/Vehicle records | Java 17 + Spring Boot | MySQL |
| `service-request` | Request lifecycle, mechanic matching, live GPS tracking | Java 17 + Spring Boot | MySQL + Firebase RTDB |
| `service-execution` | Diagnostic registration and request closure | Java 17 + Spring Boot | MySQL |
 
**Communication pattern:**
- **Sync:** REST between `api-gateway` and each service; REST from `service-request` to
  `auth-service` to validate a Driver and their active Vehicle before creating a request.
- **Async:** none between services — there is no message broker (ADR-004). The only
  asynchronous channel is FCM push from `service-request` to the driver's device.
- **Gateway:** all external requests enter through `api-gateway`, which validates the JWT
  issued by `auth-service` before routing.
```

### `deployment.md` ⭐
How the system is deployed in each environment.
**Fill in:** infrastructure diagram, what goes in Docker/K8s, network configuration, hardware requirements.

### `cross-cutting.md`
Concerns that apply to all microservices.
**Fill in:** standard logging, distributed tracing, centralized configuration, feature flags,
error handling, retry policies.

### `pattern-guide.md`
Catalog of design patterns used in the project.
**Fill in:** for each pattern: name, when to use it, when NOT to use it, concrete example from the project.

### `security-threat-model.md`
Security threat analysis of the system.
**Fill in:** using the STRIDE methodology: Spoofing, Tampering, Repudiation, Information Disclosure,
Denial of Service, Elevation of Privilege. For each threat: implemented mitigation.

### `decisions/` ⭐⭐ — Architecture Decision Records (ADRs)

One ADR per significant technical decision.
 
**ADRs in this repository:**
```
ADR-001-idioma-documentacion.md   → Why all documentation is written in English
ADR-002-architectural-style.md    → Why three microservices instead of a modular monolith
ADR-003-data-strategy.md          → Why MySQL per service + Firebase RTDB only for live GPS
ADR-004-no-message-broker.md      → Why FCM instead of Kafka or RabbitMQ
```
 
An ADR is **immutable** once accepted. If a decision changes, write a new ADR that
supersedes the old one — never edit the original.

---

## Correlations with other sections

| This section depends on... | Why |
|---|---|
| `02-domain/domain-map.md` | Bounded contexts define the service boundaries — service names must match exactly |
| `04-requirements/non-functional.md` | NFR-01 and NFR-03 are what justify the architectural style in ADR-002 |
 
| This section feeds... | Why |
|---|---|
| `06-data/models.md` | ADR-003 fixes the database engine each service uses |
| `09-microservices/service-catalog.md` | Service names, ports, and engines come from here |

---

## The 5 most common architecture mistakes

1. **Microservices too small** — If a "service" cannot exist independently, it is not a microservice.
2. **Shared database** — Destroys service independence. Each service, its own DB.
3. **Only synchronous communication** — For non-urgent operations, async events scale better.
4. **No API Gateway** — Exposing microservices directly to the frontend creates coupling.
5. **No documented decisions** — In 6 months nobody remembers why X was chosen.

---

## Questions this section must answer
 
**How is the system organized into large blocks?**
Three microservices behind an API Gateway — `auth-service`, `service-request`,
`service-execution` — each owning its own database, communicating over REST. See
`overview.md`, sections 2-4, and ADR-002.
 
**Why was each key technology chosen?**
MySQL for database-level enforcement of domain invariants; Firebase RTDB only for the
ephemeral GPS stream; FCM instead of a broker because every async need is a device
notification. See ADR-003 and ADR-004.
 
**What alternatives were evaluated and why were they discarded?**
Modular monolith and a two-service split (ADR-002); PostgreSQL and Firestore-only
(ADR-003); Kafka and RabbitMQ (ADR-004). Each ADR carries its own "Evaluated
alternatives" table with the reason for discarding.
 
**How is the system deployed?**
Not yet defined — deployment belongs to `10-devops/`, outside the scope of this
remediation.
 
**What patterns does the team apply and how?**
API Gateway and Database-per-Service are adopted. Saga, Outbox, Event Sourcing, and
Circuit Breaker are explicitly **not** adopted, because ADR-004 rules out the message
broker they depend on. Ports & Adapters is applied concretely to `service-request` in
`hexagonal-architecture.md`.