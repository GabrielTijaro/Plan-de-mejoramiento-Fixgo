# Navigation Map

> Defines the screen structure of the system, how screens connect to each other, and what routes
> exist. It is the reference when frontend and backend discuss what endpoints exist
> or how to reach a feature.

---

## Frontend route structure

```
/                                 → Splash / landing (Flutter entry point)
│
├── /auth
│   ├── /login                    → Authentication form (Firebase Auth)
│   ├── /register                 → New account — role selection: Driver or Mechanic
│   └── /forgot-password          → Password recovery
│
├── /driver                       → role: DRIVER
│   ├── /dashboard                → CTA "Request assistance" + active request card
│   ├── /vehicles                 → Vehicle list (RF2.1)
│   │   ├── /new                  → Register vehicle (plate, brand, model, type)
│   │   └── /:vehicleId/edit      → Edit / deactivate (RF2.4)
│   ├── /requests/new             → Create service request (select vehicle + location)
│   ├── /requests/:requestId/tracking → Live map, mechanic ETA (RF4.2, HU-08)
│   └── /requests/history         → Past requests + linked Diagnostic (service history)
│
├── /mechanic                     → role: MECHANIC
│   ├── /dashboard                → Incoming request queue (only if verified + AVAILABLE)
│   ├── /requests/:requestId      → Accept / reject, update status (PENDING→...→IN_PROGRESS)
│   └── /requests/:requestId/diagnostic → Close request: submit findings + finalStatus (HU-10)
│
├── /admin                        → role: ADMIN
│   ├── /mechanics/pending        → Verification queue (HU-03)
│   ├── /mechanics/:mechanicId    → Approve / reject a mechanic
│   └── /audit-logs               → Filter by date, user, module (HU-13, RF7.1)
│
└── /profile                      → All roles — language ES/EN + light/dark theme (RF6, HU-12)
```

---

## Screen map

| Screen | Route | Minimum role | Backend service |
|--------|-------|--------------|------------------|
| Login | `/auth/login` | Public | `auth-service` |
| Register | `/auth/register` | Public | `auth-service` |
| Driver dashboard | `/driver/dashboard` | DRIVER | `service-request` |
| Vehicle list | `/driver/vehicles` | DRIVER | `auth-service` |
| Register vehicle | `/driver/vehicles/new` | DRIVER | `auth-service` |
| Create service request | `/driver/requests/new` | DRIVER | `service-request` |
| Live tracking | `/driver/requests/:requestId/tracking` | DRIVER | `service-request` (Firebase RTDB) |
| Request history | `/driver/requests/history` | DRIVER | `service-execution` |
| Mechanic dashboard | `/mechanic/dashboard` | MECHANIC | `service-request` |
| Request detail (mechanic) | `/mechanic/requests/:requestId` | MECHANIC | `service-request` |
| Submit diagnostic | `/mechanic/requests/:requestId/diagnostic` | MECHANIC | `service-execution` |
| Mechanic verification queue | `/admin/mechanics/pending` | ADMIN | `auth-service` |
| Audit logs | `/admin/audit-logs` | ADMIN | `auth-service` |
| Profile / settings | `/profile` | DRIVER, MECHANIC, ADMIN | `auth-service` |

---

## Main user flows

### Flow 1 — [Name of main flow]

### Flow 1 — Create and track a service request (core flow)
 
```
Driver dashboard (/driver/dashboard)
    │
    ▼ Tap "Request assistance"
Create request (/driver/requests/new) — select active Vehicle + confirm location
    │
    ├── No active Vehicle ──► Redirect to /driver/vehicles/new (INV-005)
    │
    ▼ Submit
Live tracking (/driver/requests/:requestId/tracking) — status: PENDING
    │
    ├── Matched within SLA ──► status: ACCEPTED → ON_THE_WAY → IN_PROGRESS (push notifications, HU-11)
    │
    └── Driver cancels (PENDING/ACCEPTED only, RF3.4) ──► status: CANCELLED
    │
    ▼ Mechanic submits diagnostic
Request closed ──► status: COMPLETED, appears in /driver/requests/history
```
 
**Related HUs:** HU-06, HU-07, HU-08, HU-09, HU-11
 
### Flow 2 — Authentication
 
```
Landing (/)
    │
    ▼ Tap "Sign in"
Login (/auth/login)
    │
    ├── Valid credentials ──► Role-based dashboard (/driver/dashboard or /mechanic/dashboard)
    │
    └── Invalid credentials ──► Login with generic error (does not reveal which field failed)
```
 
**Related HUs:** HU-01, HU-02
 
### Flow 3 — Mechanic accepts and closes a request
 
```
Mechanic dashboard (/mechanic/dashboard) — only visible if status = AVAILABLE and verified (INV-001)
    │
    ▼ Open an incoming request
Request detail (/mechanic/requests/:requestId)
    │
    ▼ Accept ──► status: ACCEPTED → ON_THE_WAY → IN_PROGRESS
    ▼ On site, service finished
Submit diagnostic (/mechanic/requests/:requestId/diagnostic) — findings + finalStatus
    │
    ▼ Cannot submit without required fields (INV-004: no COMPLETED without a Diagnostic)
Request closed ──► status: COMPLETED, Mechanic returns to AVAILABLE
```
 
**Related HUs:** HU-07, HU-10
  

---

## Navigation rules

| Rule | Description |
|------|-------------|
| Authentication | Routes under `/driver`, `/mechanic`, `/admin`, `/profile` redirect to `/auth/login` if no active session |
| Authorization | `/driver/*` requires role DRIVER; `/mechanic/*` requires role MECHANIC and status ≠ verification-pending; `/admin/*` requires role ADMIN — any mismatch redirects to the user's own dashboard |
| Mechanic gating | `/mechanic/dashboard` shows the request queue only if the mechanic's own status is `AVAILABLE` (INV-001, INV-002) |
| 404 | Undefined routes show a 404 screen with a link back to the role's dashboard |
| Confirmation | Cancelling a request or deactivating a vehicle shows a confirmation dialog before executing (RF2.4, RF3.4) |

---

## Correlations

- Design system (visual components, status badge colors) → `12-ux-ui/design-system.md`
- Entities and states shown on screen → `02-domain/entities-and-rules.md`
- Roles and permissions → `00-governance/security-policy.md`
