# 01 — Project Context

> **What is this?** The "why" of the system. Anyone new must be able to read this
> folder and understand what problem the project solves, what it includes, and what it does NOT include.

## Why this section exists

Before designing anything, the team needs to agree on:
- What problem are we solving?
- For whom?
- What is in scope and what is out of scope?
- What does each term we use mean?

Without this, each team member works with different assumptions and the project fragments. 

---

## What is here and how to fill it in

### `overview.md` ⭐
Executive description of the system in maximum 1 page.
**Fill in:** system name, problem it solves, main users, key technologies,
current status (under construction / in production / legacy).

**Suggested format:**
```markdown
## What is FixGo?
FixGo is a real-time mobile platform designed to instantly connect stranded drivers with nearby mechanics and roadside assistance workshops. It streamlines emergency vehicular repairs by offering precise geolocation matching and secure digital request handling.

## Problem it solves
Drivers experiencing unexpected mechanical breakdowns face severe delays, lack reliable ways to find nearby open workshops, and struggle to get trusted roadside assistance quickly. Mechanics also miss out on local clients due to poor digital visibility and inefficient manual dispatching.

## Main users
- Drivers: Request immediate roadside assistance, track mechanic location in real-time, and manage service profiles.
- Mechanics: Receive emergency service requests, accept jobs based on proximity, and manage on-site diagnostic offerings.

## Technology stack
- Backend: Java / Maven
- Database: Firebase Realtime Database / SQL
- Infrastructure: Cloud-based deployment with AES-256 security standards
```

### `scope.md` ⭐
System boundaries: what it does and what it does NOT do.
**Fill in:** explicit list of what is INSIDE and OUTSIDE the MVP scope and future versions.
This prevents scope creep (the system that grows without control).

**Format:**
```markdown
## In scope (MVP)
- Driver and mechanic profile registration with Firebase authentication.
- Real-time geolocation tracking with a 15-meter accuracy target.
- Automated service request matchmaking with a 3-second SLA response.
- Client data protection using AES-256 encryption standards.

## Out of scope (MVP)
- In-app payment processing and financial transactions.
- Direct sale of automotive parts or physical merchandise.
- Direct employment or contracting of mechanics by FixGo.
- Provisioning of physical tow trucks by the platform.


## Candidates for future versions
- Integrated payment gateways credit cards, digital wallets.
- In-app chat and VoIP calling between drivers and mechanics.
- A user rating and review system for mechanics and workshops.
- Scheduled maintenance and preventive diagnostic bookings.
```

### `glossary.md` ⭐
Dictionary of the project domain.
**Fill in:** all technical and business terms used in the project, with their exact definition.
If two people define "client" differently, the system will have bugs.

**Format:**
```markdown
| Term | Definition | Synonyms | Notes |
|------|-----------|----------|-------|
| Driver | Individual experiencing a breakdown who requests immediate assistance. | Client, User | Initiates the matchmaking process. |
| Mechanic | Automotive professional offering on-site diagnostics and repair services. | Workshop, Provider | Must pass system verification. |
| Matchmaking | The algorithmic process of pairing a driver's request with the nearest mechanic. | Pairing, Dispatch | Must adhere to the 3-second SLA. |
| On-site Repair | Mechanical assistance provided directly at the vehicle's breakdown location. | Roadside Assistance | The core service facilitated by FixGo. |
| SLA | Service Level Agreement defining the performance benchmark of the system. | Target Response | Set at a maximum of 3 seconds for system queries. |
```

### `_template-project-profile.md`
Project technical sheet for internal records.
**Fill in:** when the project is formalized (official name, tech lead, dates, stakeholders).

### `_template-scope-declaration.md`
Formal scope declaration template for presentations or deliverables.

---

## Correlations with other sections

| If you change this... | Also review... |
|---|---|
| The problem described in `overview.md` | `03-product/vision.md` |
| The scope in `scope.md` | `04-requirements/`, `03-product/` |
| A term in `glossary.md` | Every document that uses that term — start with `02-domain/domain-map.md` |

---

## Recommended fill order

1. `overview.md` — 30 minutes with the full team
2. `scope.md` — 1 hour of discussion (the most valuable thing you can do at the start)
3. `glossary.md` — grows throughout the project, start with 10 key terms

---

## Questions this section must answer

- What does this system exist for?
- Who are the users?
- What does the system NOT do?
- What does [term X] mean in this project?
