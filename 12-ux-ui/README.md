# 12 — UX/UI

> **What is this?** The user experience design: how the system looks, how it is navigated,
> and how it behaves from the end user's perspective.

## Why design comes before code

Changing a wireframe takes 5 minutes. Changing the code takes hours.
Changing the code in production with real users can cost days and reputation.

**Design first → implement later.**

---

## What is here and how to fill it in

### `navigation-map.md` ⭐ (Start here)
The map of all screens/pages and how they connect.
**Fill in:** navigation tree, from which screen you reach which, what role can access what.

**Format:**
```markdown
## Navigation map
 
### Public area no authentication
- / (splash)
  - /auth/login
  - /auth/register
  - /auth/forgot-password
 
### Private area — Role: DRIVER
- /driver/dashboard
  - /driver/vehicles
    - /driver/vehicles/new
    - /driver/vehicles/:vehicleId/edit
  - /driver/requests/new
  - /driver/requests/:requestId/tracking
  - /driver/requests/history
  - /profile
 
### Private area — Role: MECHANIC
- /mechanic/dashboard
  - /mechanic/requests/:requestId
  - /mechanic/requests/:requestId/diagnostic
  - /profile
 
### Private area — Role: ADMIN
- /admin/mechanics/pending
  - /admin/mechanics/:mechanicId
- /admin/audit-logs
 
## Access matrix
| Screen | DRIVER | MECHANIC | ADMIN |
|--------|--------|----------|-------|
| /driver/dashboard | ✅ | ❌ | ❌ |
| /mechanic/dashboard | ❌ | ✅ | ❌ |
| /admin/mechanics/pending | ❌ | ❌ | ✅ |
```

### `wireframes.md`
Low-fidelity ASCII structure of the four screens that carry FixGo's core flow: create a
service request, live tracking, the mechanic's request detail, and mechanic verification.

### `design-system.md`
FixGo's design tokens: color palette (light/dark theme, RF6.2), typography, spacing, and
the base components reused across screens — the status badge (mapped to
`ServiceRequest`/`Mechanic` states), buttons, forms, and the admin data table.

**Format:**
```markdown
## Design tokens
 
### Colors
| Token | Value | Use |
|-------|-------|-----|
| --color-primary-500 | #1976D2 | App bar, primary buttons |
| --color-success | #388E3C | COMPLETED, verified mechanic |
| --color-error | #D32F2F | CANCELLED, rejected mechanic |
 
### Typography
| Level | Size | Weight | Use |
|-------|------|--------|-----|
| H1 | 28px | 700 | Screen titles |
| Body | 16px | 400 | General text |
 
## Components
### Status badge
Colored pill for ServiceRequest.status / Mechanic.status — see design-system.md for the
full state → color mapping.
 
### Data table Admin only
Columns, pagination, mandatory date-range filter (RF7.1) — used on /admin/audit-logs.
```
 
---

## Correlations with other sections

| This section is fed by... | And feeds into... |
|---------------------------|-------------------|
| `04-requirements/user-stories.md` → what flows exist | Screens implementing each HU |
| `02-domain/entities-and-rules.md` → what data to display | Fields in wireframes |
| `00-governance/security-policy.md` → who can access what | Access matrix in `navigation-map.md` |
---

## Questions this section must answer

- How many screens does the system have?
- How does each type of user navigate?
- What visual components are repeated?
- What is the system's visual language?
