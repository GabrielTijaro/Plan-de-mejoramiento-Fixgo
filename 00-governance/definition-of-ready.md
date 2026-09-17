# Definition of Ready (DoR)

> A User Story is **Ready** when the entire team can start it in the next sprint
> without needing to resolve fundamental questions mid-sprint.
> If a story doesn't meet this DoR, it goes back to refinement.

---

## DoR checklist

Before moving a User Story to "Ready for Sprint", verify:

# Definition of Ready (DoR) — Before Starting a Section

> A section is ready to write when the following is true. This prevents starting
> `05-architecture` before `02-domain` is stable, which is exactly how the team repo ended
> up with three different names for the same microservice.

## Checklist

- [ ] The SRS module(s) this section depends on have been re-read
- [ ] Any upstream section it depends on is already committed (see dependency order below)
- [ ] If the section introduces a new term, entity, or service name, it does not already
      exist under a different name in `01-context/glossary.md` or `02-domain/domain-map.md`

## Section dependency order for this repository

```
00-governance → 01-context → 02-domain → 03-product → 04-requirements → 05-architecture → 06-data
```

Do not start a later section using a name, entity, or decision that the earlier section
has not yet fixed — update the earlier section first.

## Correlations

- Definition of Done → `00-governance/definition-of-done.md`

---

## Common reasons a story is NOT ready

| Problem | What to do |
|---------|-----------|
| Unclear requirements | Schedule a 30-min refinement session with the PO |
| Missing acceptance criteria | PO adds criteria before the next sprint |
| Unknown dependencies | Tech Lead reviews and documents dependencies |
| Too large (> 8 SP) | Break it down into smaller stories |
| No access to test environment | DevOps generates credentials before sprint |
| Unclear API contract | Agree on contract (OpenAPI) before starting |

---

## DoR vs DoD

| | Definition of Ready (DoR) | Definition of Done (DoD) |
|-|--------------------------|--------------------------|
| **When** | Before starting the story | After finishing the story |
| **Who verifies** | Team in planning/refinement | Team in review |
| **Purpose** | Ensure the team can start without blockers | Ensure the increment is shippable |

---

## Correlations
- Definition of Done → `00-governance/definition-of-done.md`

