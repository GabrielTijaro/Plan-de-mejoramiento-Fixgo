# Product Vision

> The vision is the team's north star. All sprints, design decisions,
> and trade-offs are evaluated against this vision.
> It must be ambitious yet achievable, inspiring but specific.

---

## Vision statement

> Use the Geoffrey Moore template:

**For** stranded drivers and vehicle owners experiencing unexpected mechanical breakdowns on the road
**who** struggle with severe delays, unreliable real-time tracking, and manual unverified dispatching,
**the** **FixGo** platform
**is a** real-time roadside assistance digital ecosystem
**that** instantly connects drivers with nearby verified mechanics and workshops leveraging GPS tracking and a strict 3-second SLA response benchmark,
**unlike** traditional random directory phone calls or unverified local contacts,
**our product** guarantees automated real-time matchmaking, secure Firebase authentication, and encrypted data protection under AES-256 standards.

---

## Team mission

FixGo exists to eliminate the stress and vulnerability of roadside vehicular emergencies
by delivering an instantaneous, reliable, and secure digital dispatching experience. We
seek to modernize local workshop operations and empower drivers with instant roadside
support.

---

## Strategic pillars

| Pillar | Description | Success metrics |
|--------|-------------|----------------|
| **Speed (SLA)** | Instantaneous matching and tracking performance | Response time under 3 seconds (NFR-01) |
| **Reliability** | Consistent uptime and accurate GPS pairing | 15-meter coordinate accuracy (NFR-02) |
| **Security** | Safe user authentication and confidential data handling | 100% AES-256 and JWT compliance |
---

## High-level roadmap

> This roadmap describes FixGo's product horizon beyond the current 3-checkpoint
> documentation remediation (Entrega 1/2/3, 14-18 sep 2026 -- see
> `00-governance/agile-conventions.md`). It assumes future development sprints once
> the documentation phase is complete and code repositories exist.

```
Q1 2024 ──── Q2 2024 ──── Q3 2024 ──── Q4 2024
     │              │              │              │
  [MVP]      [Feature A]    [Feature B]   [Scale]
  Validate    Expand         Deepen        Grow
  hypothesis  the market     the value
```

| Horizon | Period | Objective | Epics / Features | Uncertainty |
|---------|--------|----------|----------------|-------------|
| H1 (Now) | Development Sprint 1-3 | MVP Launch | Driver/Mechanic auth, GPS coordinate validation, core matchmaking | Low |
| H2 (Next) | Development Sprint 4-6 | Iteration & Optimization | Real-time diagnostic status, session logs, 3s SLA performance tuning | Medium |
| H3 (Later) | Post-course | Scale & Expansion | Advanced workshop analytics, multi-city rollout, payment gateways | High |

---

## Product principles

1. **Safety First:** Every feature must prioritize the physical security of stranded drivers and transparency during on-site repairs.
2. **Performance Obsession:** Real-time tracking and dispatching must never breach the 3-second SLA benchmark (NFR-01).
3. **Simplicity in Chaos:** Emergency interfaces must be clean and usable within seconds during high-stress situations.

---

## Product Definition of Done

> The product is "done" when it achieves these OKRs:

**Objective:** [What we want to achieve]

| Key Result | Baseline | Target | Date |
|------------|---------|--------|------|
| KR1: [specific metric] | [current value] | [target value] | [date] |
| KR2: [metric] | [current] | [target] | [date] |
| KR3: [business metric] | [current] | [target] | [date] |

---

## Correlations

- Problem framing (the why) → `03-product/problem-framing.md`
- Backlog that implements the vision → `04-requirements/user-stories.md`
- KPIs in operations → `13-operations/README.md`
  