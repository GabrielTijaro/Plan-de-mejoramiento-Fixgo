# Agile Team Conventions

> Defines how the team works through its development cycles. Agree on and sign off
> with the entire team before the first sprint. Update when the team decides to change something.

---

## Sprint structure

| Checkpoint | Due | Scope |
|-------------|------------------------|--------------------------------------------------------|
| Delivery 1 | 14-sep-2026, 18:00 | Sections 00-governance, 01-context, 02-domain |
| Delivery 2 | 16-sep-2026, 18:00 | Sections 03-product, 04-requirements (based on the HU loaded by the instructor in the GitHub Project) |
| Delivery 3 | 18-sep-2026, 18:00 | Sections 05-architecture, 06-data, 12-ux-ui |
 
---
 
## Ceremonies
 
### Sprint Planning
- **When:** First day of each checkpoint — 18:00, right after the previous delivery closes
- **Duration:** Maximum 30 min (checkpoints are 2 days, not 2 weeks)
- **Who:** Individual — scope is reviewed against the instructor's assignment
- **Goal:** Select which documents belong to this checkpoint and confirm their dependencies are already committed
- **Output artifact:** The checkpoint's document list, tracked in the GitHub Project linked to this repository
### Daily Stand-up
- **When:** Not held — this is individual work, so there is no one to synchronize with
- **Replaced by:** A self-check at the end of each working session against three questions:
  1. What did I complete and commit?
  2. What is next?
  3. Is anything blocking me (e.g. waiting on the instructor's HU list)?
- **Rule:** Every session ends with a real commit, not a "WIP" placeholder
### Sprint Review
- **When:** On each checkpoint's due date — 18:00
- **Duration:** Maximum 20 min
- **Who:** Apprentice + instructor (Javier Humberto Pinto) as reviewer on the Pull Request
- **Goal:** Show the documents completed in this checkpoint and collect feedback through PR review comments
### Sprint Retrospective
- **When:** After each checkpoint review
- **Duration:** Maximum 15 min
- **Format:** What went well / What to improve / Action commitments
- **Rule:** Each retro produces at least 1 improvement action with an owner and due date
**Retro action from the 11-sep-2026 evaluation (carried into this repository):**
 
| Action | Owner | Due |
|---|---|---|
| Open the Pull Request to `main` as soon as 3 sections are complete — the team repository lost points because all work stayed on the branch and `main` was left empty | Gabriel Tijaro Jimenez | Delivery 1 |
| Never cite a document that does not exist — write the ADR first, then reference it | Gabriel Tijaro Jimenez | Delivery 3 |
 
### Backlog Refinement
- **When:** Mid-checkpoint, before starting the next section
- **Duration:** Maximum 20 min
- **Goal:** Confirm the next section's upstream dependencies are stable (see the dependency order in `definition-of-ready.md`)
- **Exit criterion:** The user story meets the Definition of Ready

---

## Estimation

### Scale
| Points | Meaning |
|--------|---------|
| 1 | Trivial — done in hours |
| 2 | Small — done in one day |
| 3 | Medium — takes 2–3 days |
| 5 | Large — takes almost a full sprint |
| 8 | Very large — should be split |
| 13 | Epic — MUST be split before the sprint |

**Technique:** T-shirt sizing mapped onto the point scale above — with a single estimator,
Planning Poker (which exists to surface disagreement between people) has nothing to compare.
**Tool:** GitHub Projects — the same board the instructor uses to load the HUs.

### Estimation rule
- If there is disagreement of 2+ levels (e.g., someone says 3 and another says 8), discuss before voting again.
- If a story is estimated at 8 or 13, it must be split into smaller sub-tasks.

---

## Backlog tool

***Tool:** GitHub Projects — the instructor loads the User Stories (HU) directly into a
GitHub Project linked to this repository. This repository's `04-requirements/user-stories.md`
must mirror the HU that appear there, not invented ones.

### Board columns
| Column | Meaning |
|--------|---------|
| Backlog | Pending refinement |
| Ready | Ready to enter the sprint (meets DoR) |
| In Progress | Someone is actively working on it |
| In Review | In Pull Request / code review |
| Done | Meets DoD and is closed |

---

## Team velocity

| Sprint | Story points completed | Notes |
|--------|----------------------|-------|
| Sprint 1 | — | — |
| Sprint 2 | — | — |
| Sprint 3 | — | — |
| Average | — | — |

---

## Related documents

- Definition of Ready → `00-governance/definition-of-ready.md`
- Definition of Done → `00-governance/definition-of-done.md`
- Risk management → `15-project-control/risks.md`
- Technical debt backlog → `15-project-control/tech-backlog.md`
