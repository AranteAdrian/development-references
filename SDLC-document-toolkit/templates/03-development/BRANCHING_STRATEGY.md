# Branching Strategy

| Field | Value |
|-------|-------|
| **Team** | [Team Name] |
| **Strategy** | [GitFlow / GitHub Flow / Trunk-Based] |
| **Last Updated** | [DD Month YYYY] |

---

## 1. Branch Types
| Branch | Pattern | Purpose | Lifetime |
|--------|---------|---------|----------|
| main | `main` | Production code | Permanent |
| develop | `develop` | Integration branch | Permanent |
| feature | `feature/TICKET-description` | New features | Until merged |
| bugfix | `bugfix/TICKET-description` | Bug fixes | Until merged |
| hotfix | `hotfix/TICKET-description` | Urgent prod fixes | Until merged |
| release | `release/vX.Y.Z` | Release prep | Until merged |

## 2. Merge Strategy
- Feature → develop: **Squash merge**
- Develop → main: **Merge commit**
- Hotfix → main: **Merge commit** (then backport to develop)

## 3. PR Requirements
- [ ] Linked to ticket
- [ ] Tests pass
- [ ] Linter passes
- [ ] 1+ reviewer approval
- [ ] No merge conflicts

## 4. Release Process
1. Create `release/vX.Y.Z` from develop
2. Final testing on release branch
3. Merge to main with tag
4. Merge back to develop
