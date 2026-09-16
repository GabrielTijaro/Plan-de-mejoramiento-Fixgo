# Problem Framing — Problem Definition

> **Why this document exists:** Before designing solutions, the team must be
> aligned on the problem it solves. This document captures that alignment.
> A well-defined problem is already halfway to a solution.

---

## 1. The problem in one sentence
**Stranded drivers experiencing unexpected vehicle breakdowns on the road** who **need immediate mechanical support** struggle with **severe delays, lack of reliable real-time tracking, and inefficient manual dispatching** because **current alternatives rely on random unverified phone calls and directory listings**, resulting in **wasted time, financial loss, and severe frustration**.

---

## 2. Affected users

 Segment | Description | Estimated size | Priority |
|---------|-------------|---------------|---------|
| **Stranded Drivers** | Vehicle owners facing unexpected mechanical failures on highways or urban roads. | High | High |
| **Local Mechanics / Workshops** | Independent mechanics and workshop operators seeking reliable client acquisition. | Medium | High |

### Jobs-to-be-done (JTBD)

**When** my vehicle breaks down unexpectedly on the road,
**I want** to instantly locate, contact, and dispatch the nearest verified mechanic with real-time tracking,
**so that** I can resume my journey safely and minimize downtime.
---

## 3. Evidence of the problem

> The problem must be real. Document the evidence you have.

| Evidence type | Source | Date | Key finding |
|--------------|--------|------|------------|
| Direct observation | Field research in local transit routes | 2026 | Average wait time for manual roadside help exceeds 45 minutes without tracking. |
| Benchmarking | Market analysis of local directory listings | 2026 | No centralized real-time platform exists for immediate local mechanical dispatching. |

---

## 4. Current user solution (and its problems)

> How does the user solve the problem today?

| Current solution | Limitations | Cost/Friction |
|-----------------|------------|--------------|
| Random phone calls to directory listings | Slow, unverified availability, no GPS tracking | High time consumption and uncertainty |

---

## 5. Solution hypothesis

> This is the first draft of the solution direction. It is not a commitment.

**We believe that** an automated real-time geolocation matching platform (FixGo)
**for** stranded drivers and local mechanics,
**will achieve** instant dispatching under a 3-second SLA response target (NFR-01).
**We will know we succeeded when** matching success rate reaches over 95% within the SLA threshold.

---

## 6. Success metrics (North Star)

| Metric | Current baseline | 6-month target | How to measure it |
|--------|----------------|---------------|-------------------|
| **Average Match Time** | 45 minutes | < 3 seconds (NFR-01) | System timestamp logs |
| **User Satisfaction** | Low (fragmented) | > 90% positive | Post-service rating app |

---

## 7. Hypothesis risks

| Risk | Probability | Impact | Experiment to validate |
|------|------------|--------|----------------------|
| [Users will not adopt the change] | High | High | [Pilot with N users] |
| [The problem is not as frequent as we think] | Medium | High | [Support log analysis] |

---

## 8. Out of scope (we do not solve)

> Explicitly define which related problems you are NOT solving in this version.
> This prevents scope creep.

- [Related problem that is out of scope: why]
- [Feature users ask for but we are not doing now: reason]

---

## Correlations

- Vision and strategic pillars -> `03-product/vision.md`
- Formal, measurable version of the 3-second SLA and matching success target -> `04-requirements/non-functional.md`
- Entities involved in matching (Driver, Mechanic, ServiceRequest) -> `02-domain/entities-and-rules.md`
