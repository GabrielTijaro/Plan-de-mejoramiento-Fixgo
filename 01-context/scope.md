# System Scope

## In Scope

What the system **DOES build and maintain**:

### MVP Features

| # | Feature | Description | SRS Module |
|---|---|---|---|
| 1 | User registration & authentication | Registration, login, roles (Driver/Mechanic/Admin), password recovery | Module 1 |
| 2 | Vehicle management | Register, update, deactivate a vehicle; associate it to a service request | Module 2 |
| 3 | Service request | Create, update, cancel a roadside-assistance request; estimated cost | Module 3 |
| 4 | Real-time tracking | Live status and mechanic location tracking, service history | Module 4 |
| 5 | Platform sync | Multi-device access, offline mode | Module 5 |
| 6 | Personalization | Language (ES/EN) and light/dark theme | Module 6 |
| 7 | Audit | Admin can query audit logs by date, user, action, module | Module 7 |

### Included integrations

| External system | Integration type | Purpose |
|-----------------|------------------|---------|
| Firebase Authentication | SDK | User authentication, session management, and JWT token generation |
| Google Maps API | REST API / SDK | Georeferencing for real-time tracking of mechanics and driver locations |
| MySQL Cloud Hosting | TCP/IP / JDBC | Cloud database for persistent relational data storage (users, vehicles, requests) |

### Environments being built

| Environment | Purpose |
|-------------|---------|
| Local | Development on the developer's machine |
| Development (dev) | Continuous integration and development testing |
| Staging | Pre-production, PO acceptance testing |
| Production | Production environment |

---

## Out of Scope

| # | What is out of scope | Reason |
|---|---|---|
| 1 | Physical vehicle repair | FixGo is a dispatch/matching platform, not a repair provider |
| 2 | Spare parts sales | Outside the core dispatch domain |
| 3 | Insurance integration | No standard public API available; legal complexity |
| 4 | In-app payments | Out of MVP budget; settlement happens directly between driver and mechanic |
| 5 | Voice assistant support | Not required to meet the core dispatch flow |
| 6 | Predictive AI analytics | Not required for MVP |


### What another system / team handles (and why not us)

| Feature | Who builds it | Why not us |
|---------|---------------|------------|
| Credential storage & Password hashing | Firebase | Reusing secure, proven infrastructure is safer and faster than building custom credential vaults |
| Payment processing | N/A (Direct cash/transfer) | Out of MVP scope; transaction happens physically between driver and mechanic |

---

## Scope assumptions

> These assumptions are taken to be true. If they change, the scope must be renegotiated.

| # | Assumption | Consequence if false |
|---|------------|----------------------|
| 1 | Firebase and external APIs maintain their free-tier limits during development | We would need to migrate to open-source alternatives or secure a project budget |
| 2 | Target users (drivers and mechanics) have mobile devices running Android 8.0+ | The Android application would not be accessible to the target demographic |
| 3 | Mechanics have reliable mobile data connections during service dispatches | Offline mode would need to be expanded, affecting real-time tracking accuracy |

---

## Constraints

| Type | Description |
|---|---|
| Time | MVP scope must be documented within the current SENA term / remediation checkpoints |
| Budget | $0 infrastructure budget — Firebase and Google Cloud free tiers only |
| Technology | Java + Spring Boot backend, Firebase, Android 8.0+ |
| Team | This repository is written individually as part of the remediation plan |
---

## External dependencies

| Dependency | Team / Provider | Required date | Status |
|------------|-----------------|---------------|--------|
| Firebase Project Credentials & SDK | Google Cloud | 14-sep-2026 | 🟢 Available |
| Cloud MySQL Database Provisioning | Hosting Provider | 16-sep-2026 | 🟡 In progress |
| Geolocation / Maps API Keys | Google Maps | 18-sep-2026 | 🔴 Pending | 

---

## How to update the scope

The scope can change, but the change has a process:

1. Document the proposed change in this file
2. Evaluate the impact on schedule and effort
3. Obtain approval from the Product Owner and Tech Lead
4. Update the roadmap in `03-product/vision.md`
5. Create or update HUs in `04-requirements/user-stories.md`

---

## Correlations

- Vision → `03-product/vision.md`
- Glossary → `01-context/glossary.md`
