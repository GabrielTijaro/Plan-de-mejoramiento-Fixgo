# 04 — Requirements

> **What is this?** The formal specification of what the system must do.
> Functional: what it does. Non-functional: how well it does it.

## Why this section exists

Requirements are the contract between the team and the client/stakeholder.
Without them:
- There is no way to verify whether the system is complete
- Scope changes have no baseline for comparison
- Tests have no success criterion

---

## Types of requirements

### Functional (FR)
Describe **what the system does**: functions, behaviors, data transformations.
*Example: "The system must allow the user to recover their password via email."*

### Non-functional (NFR)
Describe **how it does it**: quality, performance, availability, security.
*Example: "The system must respond in less than 200ms for 95% of requests."*

NFRs are usually harder to meet than FRs and are ignored more frequently. **They are equally important.**

---

## What is here and how to fill it in

### `functional.md` ⭐
List of all the system's functional requirements.
**Fill in:** numbered, with the module/service they belong to, source (originating HU), priority.

**Format:**
```markdown
| ID | Module | Description | Source (HU) | Priority |
|----|--------|-------------|------------|---------|
| FR-006 | service-request | The system must allow a driver to create a service request | HU-006 | High |
| FR-007 | service-request | The system must match the nearest available verified mechanic | HU-007 | High |
```

### `non-functional.md` ⭐
Quality, performance, and technical constraint requirements.
**Fill in:** by category (performance, availability, security, scalability, etc.)

**Format:**
```markdown
## Performance
| ID | Requirement | Metric | How to verify |
|----|------------|--------|--------------|
| NFR-001 | Response time | p95 < 200ms | Load test with K6 |

## Availability
| ID | Requirement | Metric | How to verify |
|----|------------|--------|--------------|
| NFR-010 | Uptime | 99.9% monthly | Production monitoring |

## Security
| ID | Requirement | Description | 
|----|------------|-------------|
| NFR-020 | Authentication | JWT with 1-hour expiration |
```

### `user-stories.md`
Formalized user stories (coming from the `03-product/` backlog).
**Fill in:** with As/I want/So that format + verifiable acceptance criteria.

### `traceability-matrix.md` ⭐
The chain that proves nothing was lost between the requirement and the test.
**Fill in:** FR → HU → test → service, plus reverse traceability (HU → FR) and a Gaps
table recording what is still uncovered.
**Format:**
```markdown
| HU | FR/NFR | Description | Test case | Status |
|----|--------|-------------|----------|--------|
| HU-IAM-001 | FR-001 | Login with email | TC-001 | ✅ |
```

### `_template-hu.md`
Template for a complete User Story with acceptance criteria.

### `_template-nfr.md`
Template for specifying non-functional requirements with their verification metrics.

---

## Correlations with other sections

| This section feeds... | Why |
|-----------------------|-----|
| `02-domain/entities-and-rules.md` | Business rules in acceptance criteria are the same invariants modeled there |
| `05-architecture/` | NFR-01 (latency) and NFR-03 (availability) drive the architectural style — see ADR-002 |
| `06-data/models.md` | NFR-03 and NFR-05 have direct data-layer implications |
| `09-microservices/` | Each HU names the service that implements it |

---

## Common mistakes to avoid

❌ *"The system must be fast"* → not measurable.
✅ *"Matching completes in ≤ 3 seconds (P95) from request creation to assignment."*
 
❌ *"The system must be secure"* → not verifiable.
✅ *"JWT access tokens expire in 1 hour; refresh tokens in 7 days, rotated on each use."*
 
❌ An HU with no acceptance criteria → nobody can tell when it is done.
✅ Every HU here carries at least two Gherkin scenarios: the happy path and an edge case.

---

## Questions this section must answer

- What must the system do for each type of user?
- With what speed, availability, and security?
- Which requirement originates each test case?
- Are all requirements covered by tests?
