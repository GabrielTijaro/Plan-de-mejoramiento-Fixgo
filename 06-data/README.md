# 06 — Data

> **What is this?** How the system stores, structures, and migrates data.
> In microservices, data management is one of the most complex challenges.

## Fundamental principle in microservices

> **Each microservice owns its own data.**

No service should access another service's database directly. If it needs data from another
service, it requests it via API or receives it via event. This principle guarantees independence.

---

## What is here and how to fill it in

### `models.md` ⭐
Data models for each microservice.
**Fill in:** ER (entity-relationship) diagram or description of collections/tables for each service.

```markdown
## Service: auth-service
**DB Engine:** MySQL 8 — decided in ADR-003-data-strategy.md
**Justification:** enforces domain invariants (unique plate, verification status) at the DB level
 
### Table: vehicles
| Field | Type | Nullable | Description | Constraints |
|-------|------|----------|-------------|-------------|
| id | CHAR(36) | No | Unique identifier | PK |
| plate | VARCHAR(10) | No | License plate | UNIQUE (INV-006) |
 
### Indexes
| Name | Fields | Type | Justification |
|------|--------|------|---------------|
| idx_vehicles_plate | plate | UNIQUE | Enforces INV-006 |
```

### `data-dictionary.md` ⭐
Exact meaning of each important field in the system.
**Fill in:** especially for fields that may be ambiguous or have business rules.

**Format:**
```markdown
| Field | Service | Table | Type | Detailed description | Possible values |
|-------|---------|-------|------|---------------------|-----------------|
| status | scheduling | schedule | ENUM | Current status of the schedule | ACTIVE, CANCELLED, PENDING |
```

### `modeling-conventions.md`
Naming and style conventions for the project's databases.
**Fill in:** naming (snake_case or camelCase), use of UUIDs vs sequential, standard timestamps,
soft delete vs hard delete, auditing (created_at, updated_at, created_by).

### `normalization-assessment.md`
Analysis of the normalization level and justification for denormalizations.
**Fill in:** for each intentional denormalization, explain why (performance, simplification).

### `migration-strategy.md`
Strategy for migrating data between schema versions.
**Fill in:** migration tool (Flyway, Liquibase, Alembic), rollback policy,
how to handle migrations with data in production.

---

## Correlations with other sections

| This section is fed by... | And feeds into... |
|---------------------------|-------------------|
| `02-domain/entities-and-rules.md` → domain entities | DB tables in `models.md` |
| `05-architecture/decisions/records/ADR-003-data-strategy.md` → DB engine decision | Engine choice in `models.md` |
 
> `07-api/`, `08-uml/`, and `09-microservices/` are part of the full framework scaffold
> for a later project phase — out of scope for this remediation (see
> `05-architecture/README.md` for the same note applied there).
 
---

## Important data decisions in microservices

### SQL or NoSQL?
here is no single answer. It depends on the service:
- **SQL** (PostgreSQL, MySQL): relational data, ACID transactions, fixed schema
- **Document** (MongoDB): hierarchical data, flexible schema, high variability
- **Key-value** (Redis): cache, sessions, high-speed temporary data
- **Time series** (InfluxDB, TimescaleDB): metrics, event logs

### How to handle consistency between services?
Without a shared database, consistency is **eventual**:
- Saga Pattern: chain of compensating transactions
- Outbox Pattern: guarantee that the event is published along with the transaction

---

## Questions this section must answer

- What data does each microservice handle?
- Why was that database engine chosen for each service?
- How is the schema updated without breaking the system?
- Who is the "owner" of each piece of data in the system?
