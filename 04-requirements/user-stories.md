# User Stories — Backlog

> **What to fill in here:** The product's User Story backlog.
> Each HU uses the standard format with Acceptance Criteria in Given/When/Then.
> Refined (Ready) HUs go to the sprint. Unrefined ones are epics or ideas.

---

## Backlog status

| Cut | Sprint | Total HUs | Refined | In progress | Completed |
|-----|--------|-----------|---------|-------------|-----------|
| Cut 1 | Entrega 1 (14-sep) | 5 | 5 | 0 | 0 |
| Cut 2 | Entrega 2-3 (16/18-sep) | 8 | 8 | 0 | 0 |

---

## Epics

| ID | Epic | Description |
|----|------|-------------|
| EP-001 | User & Vehicle Management | Registration, authentication, and vehicle records for drivers and mechanics (SRS Modules 1-2) |
| EP-002 | Dispatch & Tracking | Service request creation, matching, cancellation, and live tracking (SRS Modules 3-4) |
| EP-003 | Platform Support | Notifications, sync, personalization, and audit (SRS Modules 4-7) |

---

## User Stories

### HU-001 — Driver registration {#HU-001}

**Epic:** EP-001

> **As** an unregistered driver
> **I want** to create an account with my email and password
> **so that** I can request mechanical assistance

**Acceptance Criteria:**

```gherkin
Scenario 1: Successful registration
  Given I am on the registration screen
  When  I submit a valid email and a password of 8+ characters
  Then  my account is created with role DRIVER
  And   I am automatically authenticated

Scenario 2: Duplicate email
  Given an account already exists with my email
  When  I try to register with that same email
  Then  the system rejects the registration with a clear error message
```

**Definition of Done:**
- [ ] Code reviewed and approved
- [ ] Unit tests written
- [ ] Acceptance criteria verified (manual or automated)
- [ ] API contract updated if applicable
- [ ] Deployed to staging

| Field | Value |
|-------|-------|
| Story Points | 3 |
| Priority | Must Have |
| Target sprint | Entrega 1 |
| Assigned to | Gabriel Tijaro Jimenez |
| Status | Ready |
| Dependencies | None |
| Affected service(s) | auth-service |

---

### HU-002 — Login {#HU-002}

**Epic:** EP-001

> **As** a registered user
> **I want** to log in with my credentials
> **so that** I can access my account and its history

**Acceptance Criteria:**

```gherkin
Scenario 1: Successful login
  Given I have a registered account
  When  I submit correct email and password
  Then  I receive a JWT valid for 1 hour

Scenario 2: Invalid credentials
  Given I submit an incorrect password
  When  I try to log in
  Then  the system shows a generic error, without indicating which field failed
```

| Field | Value |
|-------|-------|
| Story Points | 2 |
| Priority | Must Have |
| Target sprint | Entrega 1 |
| Status | Ready |
| Dependencies | HU-001 |
| Affected service(s) | auth-service |

---
### HU-003 — Mechanic registration and verification {#HU-003}

**Epic:** EP-001

> **As** a mechanic
> **I want** to register with my workshop's data
> **so that** I can receive service requests once verified

**Acceptance Criteria:**

```gherkin
Scenario 1: Registration pending verification
  Given I submit valid workshop data
  When  I complete registration
  Then  my account is created with status PENDING_VERIFICATION

Scenario 2: Unverified mechanic cannot receive requests
  Given my account has status PENDING_VERIFICATION
  When  the matching algorithm looks for available mechanics
  Then  I do not appear in the results
```

| Field | Value |
|-------|-------|
| Story Points | 3 |
| Priority | Must Have |
| Target sprint | Entrega 1 |
| Status | Ready |
| Dependencies | None |
| Affected service(s) | auth-service |

---

### HU-004 — Vehicle registration {#HU-004}

**Epic:** EP-001

> **As** a driver
> **I want** to register my vehicle with plate, brand, model, and type
> **so that** I can associate it with a service request

**Acceptance Criteria:**

```gherkin
Scenario 1: Successful registration
  Given I am an authenticated driver
  When  I submit a vehicle with a unique plate
  Then  the vehicle is created with status ACTIVE

Scenario 2: Duplicate plate
  Given a vehicle already exists with that plate
  When  I try to register it again
  Then  the system rejects the registration
```

| Field | Value |
|-------|-------|
| Story Points | 2 |
| Priority | Must Have |
| Target sprint | Entrega 1 |
| Status | Ready |
| Dependencies | HU-001 |
| Affected service(s) | auth-service |

---

### HU-005 — Deactivate a vehicle {#HU-005}

**Epic:** EP-001

> **As** a driver
> **I want** to deactivate a vehicle I no longer use
> **so that** it stops appearing as available without losing its history

**Acceptance Criteria:**

```gherkin
Scenario 1: Deactivation preserves history
  Given a vehicle with past service requests
  When  I deactivate it
  Then  its past requests remain visible in my history

Scenario 2: Inactive vehicle cannot be used
  Given a vehicle with status INACTIVE
  When  I try to create a new service request with it
  Then  the system rejects the request
```

| Field | Value |
|-------|-------|
| Story Points | 1 |
| Priority | Should Have |
| Target sprint | Entrega 1 |
| Status | Ready |
| Dependencies | HU-004 |
| Affected service(s) | auth-service |

---

### HU-006 — Create a service request {#HU-006}

**Epic:** EP-002

> **As** a driver
> **I want** to create an assistance request with my vehicle and location
> **so that** a nearby available mechanic is assigned

**Acceptance Criteria:**

```gherkin
Scenario 1: Successful request
  Given I have an active vehicle and a valid GPS location
  When  I create a service request
  Then  the request starts with status PENDING
  And   the matching process is triggered immediately

Scenario 2: No active vehicle
  Given I have no active vehicle registered
  When  I try to create a service request
  Then  the system rejects the request and asks me to register a vehicle
```

| Field | Value |
|-------|-------|
| Story Points | 5 |
| Priority | Must Have |
| Target sprint | Entrega 2 |
| Status | Ready |
| Dependencies | HU-004 |
| Affected service(s) | service-request |

---

### HU-007 — Automatic matching to nearest available mechanic {#HU-007}

**Epic:** EP-002

> **As** a driver
> **I want** the system to automatically assign the nearest available mechanic
> **so that** I don't have to search for one manually

**Acceptance Criteria:**

```gherkin
Scenario 1: Match found within SLA
  Given at least one verified, available mechanic is within range
  When  a service request is created
  Then  a mechanic is matched within 3 seconds (P95, NFR-01)

Scenario 2: No mechanic available
  Given no verified mechanic is available nearby
  When  matching runs
  Then  the driver receives an explicit "no mechanic available" notification, not a generic error
```

| Field | Value |
|-------|-------|
| Story Points | 8 |
| Priority | Must Have |
| Target sprint | Entrega 2 |
| Status | Ready |
| Dependencies | HU-006 |
| Affected service(s) | service-request |

---

### HU-008 — Live tracking of the assigned mechanic {#HU-008}

**Epic:** EP-002

> **As** a driver
> **I want** to see the assigned mechanic's live location
> **so that** I know how long until they arrive

**Acceptance Criteria:**

```gherkin
Scenario 1: Live position updates
  Given a mechanic has been matched to my request
  When  the mechanic moves
  Then  I see their position updated within a 15-meter accuracy (NFR-02)

Scenario 2: Status visible without refresh
  Given my request is ON_THE_WAY
  When  the status changes to IN_PROGRESS
  Then  I see the new status without reloading the app
```

| Field | Value |
|-------|-------|
| Story Points | 5 |
| Priority | Must Have |
| Target sprint | Entrega 2 |
| Status | Ready |
| Dependencies | HU-007 |
| Affected service(s) | service-request |

---

### HU-009 — Cancel a service request {#HU-009}

**Epic:** EP-002

> **As** a driver
> **I want** to cancel a request before it is accepted
> **so that** I don't wait for a service I no longer need

**Acceptance Criteria:**

```gherkin
Scenario 1: Cancel a pending request
  Given my request has status PENDING or ACCEPTED
  When  I cancel it
  Then  its status changes to CANCELLED

Scenario 2: Cannot cancel an in-progress request
  Given my request has status IN_PROGRESS
  When  I try to cancel it
  Then  the system rejects the cancellation
```

| Field | Value |
|-------|-------|
| Story Points | 2 |
| Priority | Should Have |
| Target sprint | Entrega 2 |
| Status | Ready |
| Dependencies | HU-006 |
| Affected service(s) | service-request |

---

### HU-010 — Register a diagnostic and close a service request {#HU-010}

**Epic:** EP-002

> **As** a mechanic
> **I want** to register the diagnostic and findings when finishing the service
> **so that** there is a record of what was done

**Acceptance Criteria:**

```gherkin
Scenario 1: Closing requires a diagnostic
  Given a request with status IN_PROGRESS
  When  I submit a diagnostic
  Then  the request moves to COMPLETED

Scenario 2: Cannot close without a diagnostic
  Given a request with status IN_PROGRESS
  When  I try to mark it COMPLETED without submitting a diagnostic
  Then  the system rejects the transition
```

| Field | Value |
|-------|-------|
| Story Points | 3 |
| Priority | Must Have |
| Target sprint | Entrega 3 |
| Status | Ready |
| Dependencies | HU-007 |
| Affected service(s) | service-execution |

---

### HU-011 — Push notifications on status change {#HU-011}

**Epic:** EP-003

> **As** a driver
> **I want** to receive a notification when my request's status changes
> **so that** I don't have to keep checking the app

**Acceptance Criteria:**

```gherkin
Scenario 1: Notification on each transition
  Given my request changes status
  When  the transition is saved
  Then  an FCM push notification is sent to my device

Scenario 2: Offline delivery
  Given my device is offline when the status changes
  When  my device reconnects
  Then  I receive the pending notification
```

| Field | Value |
|-------|-------|
| Story Points | 3 |
| Priority | Should Have |
| Target sprint | Entrega 3 |
| Status | Ready |
| Dependencies | HU-007 |
| Affected service(s) | service-request |

---

### HU-012 — Language and theme personalization {#HU-012}

**Epic:** EP-003

> **As** a user
> **I want** to choose the app's language (ES/EN) and theme (light/dark)
> **so that** I can use it in my language and with the visibility I prefer

**Acceptance Criteria:**

```gherkin
Scenario 1: Preference persists
  Given I change the language to English
  When  I close and reopen the app
  Then  the app remains in English

Scenario 2: No re-login required
  Given I am logged in
  When  I change the theme
  Then  my session remains active
```

| Field | Value |
|-------|-------|
| Story Points | 2 |
| Priority | Could Have |
| Target sprint | Entrega 3 |
| Status | Ready |
| Dependencies | HU-002 |
| Affected service(s) | auth-service |

---

### HU-013 — Query audit logs {#HU-013}

**Epic:** EP-003

> **As** an administrator
> **I want** to query the audit log filtering by date, user, and module
> **so that** I can investigate incidents or disputes

**Acceptance Criteria:**

```gherkin
Scenario 1: Filtered query
  Given I provide a mandatory date range
  When  I run the query
  Then  I see matching entries with user, action, module, and result

Scenario 2: Missing date range
  Given I do not provide a date range
  When  I try to run the query
  Then  the system rejects the request to avoid unbounded scans
```

| Field | Value |
|-------|-------|
| Story Points | 3 |
| Priority | Should Have |
| Target sprint | Entrega 3 |
| Status | Ready |
| Dependencies | None |
| Affected service(s) | auth-service |

---
## Rules for writing HUs

### 1. The role matters
Do not write "As a user" — that says nothing. Use the specific role:
```
✓ As a system administrator
✓ As a registered customer
✓ As an inventory operator
✗ As a user
✗ As a person
```

### 2. The benefit justifies the work
The "so that" must describe a business benefit, not redescribe the action:
```
✓ so that I can manage my orders without calling support
✗ so that I can see my orders (this only describes the feature)
```

### 3. ACs are verifiable
Each AC must be verifiable manually or automatable as a test:
```
✓ Then the system shows a message "Order #123 confirmed"
✓ Then the confirmation email arrives in less than 30 seconds
✗ Then the system works well (not verifiable)
✗ Then the user is satisfied (not verifiable)
```

### 4. One HU = one unit of value
If the HU has 15 ACs, it is probably 3 HUs.
The team must be able to complete it in one sprint (maximum 2 weeks).

---

## Ready-to-copy HU template

```markdown
### HU-00X — [Name] {#HU-00X}

**Epic:** EP-00X

> **As** [role]
> **I want** [action]
> **so that** [benefit]

**Acceptance Criteria:**

\```gherkin
Scenario 1: [name]
  Given [context]
  When  [action]
  Then  [result]
\```

| Field | Value |
|-------|-------|
| Story Points | |
| Priority | |
| Target sprint | |
| Status | Backlog |
| Dependencies | |
```

---

## Correlations

- Full template with DoD checklist → `04-requirements/_template-hu.md`
- Non-functional requirements → `04-requirements/non-functional.md`
- Traceability matrix → `04-requirements/traceability-matrix.md`
- API contracts derived from these HUs → `07-api/contracts/openapi/`
