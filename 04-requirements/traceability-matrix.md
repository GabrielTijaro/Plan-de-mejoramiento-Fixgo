# Traceability Matrix

> Traceability connects every line of code to its business justification.
> It allows answering: "Why does this function exist?" and "Which HU covers this part of the system?"
> It also identifies: unimplemented requirements and code without a requirement (possible technical debt).

---

## How to use this matrix

```
Requirement → HU → Test Case → Implementation → Service

If a requirement has no HU: it is not planned
If a HU has no test case: it has no completeness criterion
If a test case has no implementation: there is test technical debt
If there is code without an HU: possible gold-plating or bug introduced without a story
```

---

## FR → HU → Test → Service matrix

| FR ID | FR Description | HU(s) | Tests that verify it | Service | Status |
|-------|---------------|-------|---------------------|---------|--------|
| FR-001 | System allows driver registration | HU-001 | Not written yet | auth-service | 🔴 Pending |
| FR-002 | System allows login with JWT issuance | HU-002 | Not written yet | auth-service | 🔴 Pending |
| FR-003 | System allows mechanic registration and verification | HU-003 | Not written yet | auth-service | 🔴 Pending |
| FR-004 | System allows vehicle registration | HU-004 | Not written yet | auth-service | 🔴 Pending |
| FR-005 | System allows deactivating a vehicle without losing history | HU-005 | Not written yet | auth-service | 🔴 Pending |
| FR-006 | System allows creating a service request | HU-006 | Not written yet | service-request | 🔴 Pending |
| FR-007 | System automatically matches the nearest available mechanic | HU-007 | Not written yet | service-request | 🔴 Pending |
| FR-008 | System shows live tracking of the assigned mechanic | HU-008 | Not written yet | service-request | 🔴 Pending |
| FR-009 | System allows cancelling a pending/accepted request | HU-009 | Not written yet | service-request | 🔴 Pending |
| FR-010 | System allows registering a diagnostic and closing a request | HU-010 | Not written yet | service-execution | 🔴 Pending |
| FR-011 | System sends push notifications on status change | HU-011 | Not written yet | service-request | 🔴 Pending |
| FR-012 | System allows language and theme personalization | HU-012 | Not written yet | auth-service | 🔴 Pending |
| FR-013 | System allows administrators to query audit logs | HU-013 | Not written yet | auth-service | 🔴 Pending |

---

## NFR → Validation matrix

| NFR ID | Description | How it is validated | Tool | Status |
|--------|-------------|-------------------|------|--------|
| NFR-001 | Matching SLA ≤ 3s (P95); P95 < 300ms other endpoints | Load test in staging | k6 (planned) | 🔴 Pending |
| NFR-001 | GPS tracking accuracy within 15m | Manual field test | N/A | 🔴 Pending |
| NFR-002 | 99.9% availability | SLO monitoring | Grafana (planned) | 🔴 Pending |
| NFR-004 | JWT authentication, 1h expiration | Security contract test | Postman (planned) | 🔴 Pending |

---

## Inverse traceability: HU → FR

| HU | Title | FR(s) it implements | Sprint |
|----|-------|---------------------|--------|
| HU-001 | Driver registration | FR-001 | Entrega 1 |
| HU-002 | Login | FR-002 | Entrega 1 |
| HU-003 | Mechanic registration and verification | FR-003 | Entrega 1 |
| HU-004 | Vehicle registration | FR-004 | Entrega 1 |
| HU-005 | Deactivate a vehicle | FR-005 | Entrega 1 |
| HU-006 | Create a service request | FR-006 | Entrega 2 |
| HU-007 | Automatic matching | FR-007 | Entrega 2 |
| HU-008 | Live tracking | FR-008 | Entrega 2 |
| HU-009 | Cancel a service request | FR-009 | Entrega 2 |
| HU-010 | Diagnostic and close request | FR-010 | Entrega 3 |
| HU-011 | Push notifications | FR-011 | Entrega 3 |
| HU-012 | Language and theme personalization | FR-012 | Entrega 3 |
| HU-013 | Query audit logs | FR-013 | Entrega 3 |

---

## Status legend

| Status | Meaning |
|--------|---------|
| ✅ Done | Implemented, tested, and in production |
| 🟡 In progress | Under development in the current sprint |
| 🔴 Pending | In the backlog, not started |
| ⏸ Blocked | Has an external blocker |
| ❌ Cancelled | Removed from scope |

---

## Identified gaps (requirements without coverage)

> This section is updated automatically or manually when reviewing the matrix.
> A gap is: an FR without an HU, or an HU without a test, or a test without implementation.

| Gap type | Description | Required action | Owner | Date |
|----------|-------------|----------------|-------|------|
| HU without test | All 13 HUs (HU-001 to HU-013) have Gherkin acceptance criteria but no automated test file yet | Write test files once each service's codebase exists | Gabriel Tijaro Jimenez | Post-remediation, once code repos start |
| NFR without validation tool | All NFRs are defined with metrics but no CI pipeline validates them yet | Set up k6/Grafana once `10-devops/` is built | Gabriel Tijaro Jimenez | Post-remediation |
---

## How to maintain this matrix

1. When an HU is created: add the row in the FR → HU → Test → Service section
2. When a test is written: note the file in the "Tests that verify it" column
3. When an HU is completed: change the status to ✅
4. At each Sprint Planning: review gaps and assign actions

---

## Correlations

- User Stories → `04-requirements/user-stories.md`
- Non-Functional Requirements → `04-requirements/non-functional.md`
- Testing strategy → `11-quality/testing-strategy.md`
- DoD that determines when an HU is Done → `00-governance/definition-of-done.md`  
