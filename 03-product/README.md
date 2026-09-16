# 03 — Product Definition

> **What is this?** The answer to "what are we going to build?". It is not "how" — that comes in
> architecture. Here the validated problem, product vision, and build plan are defined.

## Why this section exists

Without a clear product definition:
- The team builds features nobody asked for
- Scope grows out of control (scope creep)
- There is no way to know whether the project was successful

This section is the contract between the team and stakeholders about **what will be built and why**.

---

## What is here and how to fill it in

### `problem-framing.md` ⭐ (Start here)
Articulates the problem before proposing solutions.
**Fill in:** who has the problem, exactly what pain, evidence of the problem, how they solve it today.

**Format:**
```markdown
## The problem
**Who has it?** Stranded drivers and vehicle owners with an unexpected mechanical breakdown
**What problem do they have?** Severe delays, unreliable tracking, unverified manual dispatching
**When does it occur?** Highway or urban breakdown, no organized channel to reach a mechanic
**What is the impact?** ~45 min average wait, financial loss, frustration
**How do they solve it today?** Random unverified phone calls to directory listings

## Why it is worth solving
Eliminates the stress and vulnerability of roadside emergencies with an instant,
reliable, secure digital dispatching experience (see `vision.md`).
```

### `discovery-brief.md`
Findings from user research.
**Fill in:** interviews conducted, insights found, assumptions validated and invalidated.

### `vision.md` ⭐
For stranded drivers
and vehicle owners, who struggle with severe delays and unreliable manual dispatching,
FixGo is a real-time roadside assistance ecosystem that instantly connects them with
nearby verified mechanics under a 3-second SLA. Unlike random directory phone calls, our
product guarantees automated matchmaking with secure, encrypted authentication."


### `roadmap.md`
Delivery plan over time. In this repository, the high-level roadmap lives inside
`vision.md` instead of a separate file, since it is short enough not to need one.

**Format:**
```markdown
## Phase 1 — MVP (Development Sprint 1-3)
- Driver and Mechanic registration with Firebase Auth
- Vehicle registration and GPS coordinate validation
- Core matchmaking: assign nearest available verified mechanic
 
## Phase 2 — Iteration (Development Sprint 4-6)
- Real-time diagnostic status updates during the service
- Session and audit logs (SRS Module 7)
- Performance tuning to hold the 3-second matching SLA (NFR-01)
 
## Phase 3 — Scale (Post-course)
- Workshop analytics for mechanics
- Multi-city rollout
- Payment gateway integration (out of scope for v1.0)
```

### `product-backlog.md` ⭐
Prioritized list of everything that must be built. In this repository, this role is
filled by `04-requirements/user-stories.md` and its Epics table instead of a separate
backlog file.

### `_template-prd.md`
Complete Product Requirements Document.
**Use when:** you need to formalize requirements for an external stakeholder or academic delivery.

### `_template-discovery-brief.md`
Template for documenting user research.

### `_template-problem-framing.md`
Structured template for framing the problem.

### `_template-backlog.md`
Template for initial backlog user stories.

---

## User Story format

```markdown
> This repository's actual HU format lives in `04-requirements/user-stories.md`
> (Gherkin `Given/When/Then` acceptance criteria, IDs `HU-001` to `HU-013`) — that is the
> format to follow, not a separate one defined here.
 
### HU-0NN — Descriptive name {#HU-0NN}
 
**Epic:** EP-00X
 
> **As** a driver
> **I want** to create an assistance request with my vehicle and location
> **so that** a nearby available mechanic is assigned
 
**Acceptance Criteria:**
 
Scenario 1: Successful request
  Given I have an active vehicle and a valid GPS location
  When  I create a service request
  Then  the request starts with status PENDING
 
**Estimation:** 5 SP   **Priority:** Must Have

```

---

## Correlations with other sections

| This section feeds... | Why |
|-----------------------|-----|
| `04-requirements/user-stories.md` | Epics in the vision and roadmap become the Epics table there, then get formalized as HUs |
| `02-domain/` | Problem framing reveals the domain entities: Driver, Mechanic, Vehicle, ServiceRequest |
| `05-architecture/` | The 3-second SLA in the vision drives the architectural style decision (ADR-002) |
---

## Questions this section must answer
 
**What problem exactly are we solving?**
Stranded drivers with an unexpected vehicle breakdown have no organized, verified,
trackable way to reach a nearby mechanic — they rely on random phone calls to directory
listings, with ~45 minutes average wait and no visibility into who is coming or when
(`problem-framing.md`, section 1).
 
**What does product success look like?**
Mechanic assignment completed within the 3-second SLA (NFR-01) for over 95% of requests,
with live GPS tracking accurate to 15 meters (NFR-02) — the North Star Metric in
`problem-framing.md`, section 6.
 
**What do we build first and why?**
User and vehicle registration, then matching and live tracking — in that order, because a
service request cannot exist without an authenticated driver and an active registered
vehicle (`02-domain/entities-and-rules.md`, INV-005). This is Phase 1 of the roadmap.
 
**What do we NOT build in this cycle?**
Physical repair, spare parts sales, insurance integration, in-app payments, voice
assistants, and predictive AI analytics — all explicitly out of scope for v1.0
(`01-context/scope.md`).
