# Definition of Done (DoD)

> A User Story is **DONE** when it meets ALL criteria on this checklist.
> If even one is missing, the story is NOT done — it goes back to In Progress.

## Mandatory checklist

- [ ] The document contains real FixGo information — no `[bracketed placeholder]`, no
      leftover `> [!NOTE] INSTRUCTIONS` block, no copy-pasted citation artifacts (`[cite: N]`)
- [ ] Every claim that references the SRS cites the specific module or RF it comes from
      (e.g. "SRS RF2.1"), so a reader can verify it
- [ ] Terminology matches `01-context/glossary.md` — the same concept is not called two
      different names in two documents (this was the main issue found in the team repo:
      `dispatch-service` vs `service-request` vs `geolocation-service` for the same thing)
- [ ] The document does not contradict another already-written document (cross-check before
      committing — e.g. a database engine decided in one file must match every other file
      that mentions it)
- [ ] Committed with the `docs(section): description` format from `git-conventions.md`

---

## Allowed exceptions

The following exceptions must be explicitly agreed to by the Tech Lead:
- E2E tests omitted due to environment limitations (document the risk)
- Documentation deferred for urgent delivery (create a tech-debt ticket)

---

## What is NOT a Done criterion

- "The code is on my machine" — it must be in the repository
- "It works on my local environment" — it must work on staging
- "The PM/PO approved it" — that is the product Definition of Done, not the code's
