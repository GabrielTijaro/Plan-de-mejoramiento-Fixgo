# Project Glossary
---

## How to use this glossary

1. Before using a technical or business term in code, docs, or conversations: look it up here.
2. If it's not there: add it with its definition.
3. If there is disagreement about the definition: discuss it as a team and update this document.

---

## Domain terms

| Term | Definition |
|---|---|
| **Driver** | A registered user who owns at least one vehicle and can create service requests |
| **Mechanic** | A verified technician or workshop that receives and fulfills service requests |
| **Vehicle** | A car or motorcycle registered by a Driver (plate, brand, model, type — SRS RF2.1) |
| **Service Request** | The ticket created by a Driver describing a breakdown, its location, and status |
| **Matchmaking** | The process of assigning an available nearby Mechanic to a Service Request |
| **Diagnostic** | The record a Mechanic submits when closing a Service Request, describing the finding and outcome |
| **Verification** | The admin process that confirms a Mechanic's credentials before they can accept requests |
| **Dispatch Range** | The maximum radius within which a Mechanic is eligible to receive a request |
| **Direct Settlement** | Payment for the service completed directly between Driver and Mechanic, outside the app (in-app payments are out of scope) |
| **Audit Log** | A recorded system action (who, what, when, module, result) queryable by an Administrator per SRS Module 7 |

---

## Technical terms of the project

| Term | Definition |
|------|-----------|
| Microservice | Independent service with a single responsibility, its own process, and its own database |
| Domain Event | A fact that occurred in the business that other services can observe. Name always in past tense. |
| Bounded Context | Boundary within which a particular domain model has consistent meaning |
| API Gateway | Single entry point to the system that routes requests to the corresponding microservices |
| Circuit Breaker | Pattern that stops calls to a failing service, preventing failure cascades |
| Saga | Sequence of local transactions across different services with compensating transactions on failure |
| Dead Letter Queue | Queue where messages that could not be processed after several retries are sent |
| Idempotence | Property of an operation to produce the same result if executed multiple times |
| Bounded Context | A boundary within which a domain model has one consistent meaning |
| Domain Event | A fact that already happened in the business, named in past tense |
| JWT | JSON Web Token — used for authenticated API calls after Firebase login |
---

## Acronyms

| Acronym | Meaning |
|---|---|
| SRS | Software Requirements Specification |
| RF | Functional Requirement |
| NFR | Non-Functional Requirement |
| ADR | Architecture Decision Record |
| FCM | Firebase Cloud Messaging |