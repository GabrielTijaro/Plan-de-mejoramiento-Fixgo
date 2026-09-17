Git Conventions — FixGo 

> This is an individual documentation repository created as part of the FixGo remediation
> plan ADSO-3239188. It documents the same FixGo system as
> the team repository (`fixgo-docs`), rewritten independently by this apprentice.

## Branch strategy

```
main        ← Production. Merge from release only. Always stable.
  └── dev   ← Continuous integration. Merge from features.
        └── feat/[description]    ← One branch per feature/user story
        └── fix/[description]     ← One branch per bugfix
        └── chore/[description]   ← Infrastructure, docs, dependency changes
        └── hotfix/[description]  ← Urgent fixes directly to main
```

**Rules:**
- Nobody commits directly to `main` or `dev`
- Every task = one branch + one Pull Request
- One branch = one task (do not mix different features)
- Branches are deleted after merge

---

## Branch naming format

```
[type]/[description-in-kebab-case]

Examples:
feat/oauth2-login
fix/schedule-overlap-calculation
chore/update-spring-dependencies
hotfix/null-token-expiration
```

---

## Commit format (Conventional Commits)

This is a documentation-only sprint, so every commit uses the `docs` type. The scope is
the section number and name; the description is lowercase, imperative, no trailing period.

```
docs(section-name): lowercase description, imperative mood, no trailing period
```

**Examples used in this repository:**
```
docs(00-governance): personalize git and agile conventions for FixGo
docs(01-context): write system overview and scope
docs(01-context): add glossary with 10 domain terms
docs(02-domain): define bounded contexts and service ownership
docs(02-domain): add Driver, Vehicle, Mechanic, ServiceRequest and Diagnostic entities
docs(02-domain): add domain event catalog
```

**Do not use** `feat`, `fix`, `chore`, `refactor`, etc. in this repository — those types
belong to a code repository, not a documentation-only remediation deliverable.


**Types:**
| Type | When to use |
|------|-------------|
| `feat` | New functionality |
| `fix` | Bug fix |
| `docs` | Documentation only |
| `style` | Formatting, whitespace (no logic change) |
| `refactor` | Code refactoring without behavior change |
| `test` | Add or modify tests |
| `chore` | Tooling, dependencies, CI |
| `perf` | Performance improvement |

**Examples:**
```
feat(iam): implement JWT login

fix(scheduling): correct schedule overlap validation
Closes #42

docs(api): update actor service OpenAPI contract

chore(deps): upgrade Spring Boot to 3.2.0
```

---

## Pull Request policy

- **Size:** maximum 400 lines of code (excluding tests). If larger, split it.
- **Reviewers:** minimum 1 approval before merging
- **Review time:** reviewer has a maximum of 24 business hours
- **Template:** use the template at `.github/pull_request_template.md`
- **Green CI:** merge only proceeds if all pipeline checks pass

---

## Merge policy

- Use **Squash and Merge** for features (keeps `dev` history clean)
- Use **Merge Commit** for releases to `main` (preserves full history)
- **Do not** use Rebase & Merge (creates confusion in shared history)

---

## Tags and versioning

Follow [SemVer](https://semver.org/): `MAJOR.MINOR.PATCH`

```bash
# When releasing to production
git tag -a v1.2.0 -m "Release v1.2.0: add reports module"
git push origin v1.2.0
```
