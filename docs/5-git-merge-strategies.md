# Git Merge Strategies: Merge vs Squash vs Rebase
### A Practical Reference for Understanding When and Why to Use Each

> *Part of the Engineering Standards Series — Adrian Arante*

---

## Why This Document Exists

This document came out of a real moment of confusion.

While working through a Tier 0 milestone — a set of 8 security and stability fixes on an inherited repository — the GitHub commit history looked strange. Every task appeared twice. There were merge messages interleaved with actual commit messages. It looked like commits were duplicated and the history was impossible to read cleanly.

```
Merge fix/cors-wildcard into restructure/tier-0     ← what is this?
fix: replace CORS wildcard with explicit origin      ← the actual work
Merge fix/dummy-credential into restructure/tier-0  ← what is this?
fix: replace hardcoded Fabric server GUID            ← the actual work
...and so on for every task
```

Nothing was actually wrong. But it prompted the right question — **what is the best way to work when you have a milestone branch and you still want each task to be isolated in its own branch within that milestone, without the history becoming noisy and hard to read?**

The answer is squash merge. This document explains why, what the alternatives are, and exactly what each strategy looks like in practice so you never have to be surprised by your own git history again.

---

## Table of Contents

1. [The Confusion Nobody Talks About](#1-the-confusion-nobody-talks-about)
2. [The Three Strategies Explained](#2-the-three-strategies-explained)
   - 2.1 [Merge with --no-ff](#21-merge-with---no-ff)
   - 2.2 [Squash and Merge](#22-squash-and-merge)
   - 2.3 [Rebase and Merge](#23-rebase-and-merge)
   - 2.4 [Fast Forward (FF)](#24-fast-forward-ff)
3. [Side by Side Comparison](#3-side-by-side-comparison)
4. [What the History Actually Looks Like](#4-what-the-history-actually-looks-like)
5. [Real World Scenario — What Happened and What Should Have Happened](#5-real-world-scenario--what-happened-and-what-should-have-happened)
6. [The Milestone Branch Strategy](#6-the-milestone-branch-strategy)
7. [Solo vs Team — When to Use What](#7-solo-vs-team--when-to-use-what)
8. [Squash and the Closed PR — Nothing Is Ever Lost](#8-squash-and-the-closed-pr--nothing-is-ever-lost)
9. [How to Set Squash as Default in GitHub](#9-how-to-set-squash-as-default-in-github)
10. [Quick Reference](#10-quick-reference)

---

## 1. The Confusion Nobody Talks About

Most engineers learn Git commands without ever being shown what the history looks like after each type of merge. They merge, push, and move on — and when they open GitHub and see a confusing list of duplicate-looking entries, they think something went wrong.

Nothing went wrong. The history is just telling you more than you expected.

This document explains exactly what each merge strategy does, what the GitHub history looks like after each one, and which one to use in which situation — using a real scenario as the example throughout.

---

## 2. The Three Strategies Explained

### 2.1 Merge with --no-ff

**What it does:**
Brings all commits from the task branch onto the destination branch and creates an additional merge commit on top to record the event of the merge.

```
Before:
    milestone:  A → B → C
    task:       A → B → C → D → E

After --no-ff merge:
    milestone:  A → B → C → D → E → M
                                      ↑
                                merge commit (extra entry)
```

**What you see in GitHub history:**

```
Merge fix/cors-wildcard into restructure/tier-0   ← merge commit (extra)
fix: remove allow_credentials flag                ← commit E
fix: update allowed origins list                  ← commit D
```

Every individual commit from the task branch is visible. Plus an extra merge commit on top. If you had 8 tasks with 2 commits each, you see 8 merge commits + 16 work commits = 24 entries for what is logically 8 changes.

**When to use:**
When you explicitly want a permanent record of every branch boundary in the history. Common in open source projects where knowing which PR introduced which commit matters.

---

### 2.2 Squash and Merge

**What it does:**
Collapses all commits from the task branch into a single new commit on the destination branch. No merge commit is created. The task branch's individual commits never appear on the milestone branch.

```
Before:
    milestone:  A → B → C
    task:       A → B → C → D → E → F (3 commits including WIP)

After squash merge:
    milestone:  A → B → C → S
                              ↑
                         one clean commit
                         (D, E, F collapsed into S)
```

**What you see in GitHub history:**

```
fix: replace CORS wildcard with explicit origin   ← one clean commit
```

That is it. One entry regardless of how many commits were inside the task branch. WIP commits, intermediate fixes, experimental attempts — all hidden. Only the final clean commit message you write at squash time is visible.

**When to use:**
Solo developer or team wanting a clean readable milestone history. The most common choice for product teams.

---

### 2.3 Rebase and Merge

**What it does:**
Replays each commit from the task branch directly onto the destination branch one by one, without a merge commit. All individual commits land on the destination but in a linear sequence as if they were always there.

```
Before:
    milestone:  A → B → C
    task:       A → B → C → D → E → F

After rebase merge:
    milestone:  A → B → C → D → E → F
                              (no merge commit, but all 3 commits visible)
```

**What you see in GitHub history:**

```
fix: remove allow_credentials flag        ← commit F
fix: update allowed origins list          ← commit E
wip: initial attempt                      ← commit D (WIP visible)
```

All commits including messy WIP ones land on the milestone. No merge commit but the full working history is exposed.

**When to use:**
When you want a linear history without merge commits but still want to preserve individual commits. Common in projects where every commit represents a meaningful tested change. Less common in practice because WIP commits are typically not meaningful to others.

---

### 2.4 Fast Forward (FF)

**What it does:**
Similar to rebase — moves the milestone branch pointer forward to include all the task branch commits as if they were always there. No merge commit. All task branch commits land directly.

The difference from rebase is that fast forward only works when the milestone branch has not moved ahead of where the task branch was created. If new commits have been added to the milestone since the task branch was created, fast forward is not possible and Git falls back to a merge commit.

```
Only possible when:
    milestone:  A → B → C
    task:       A → B → C → D → E
    (milestone has not moved since task branch was created)

Not possible when:
    milestone:  A → B → C → X → Y  (other commits added)
    task:       A → B → C → D → E
    (must use merge or rebase instead)
```

**When to use:**
Simple linear workflows with no parallel development. Not practical when multiple engineers work on the same milestone simultaneously.

---

## 3. Side by Side Comparison

Say a task branch `fix/cors-wildcard` had 3 commits inside it:

```
wip: initial attempt
fix: update allowed origins list
fix: remove allow_credentials flag
```

| Strategy | Entries added to milestone | Merge commit | WIP visible | Clean history |
|---|---|---|---|---|
| `--no-ff` | 3 commits + 1 merge commit = 4 | YES | YES | NO |
| Squash | 1 commit | NO | NO | YES |
| Rebase | 3 commits | NO | YES | PARTIAL |
| Fast Forward | 3 commits | NO | YES | PARTIAL |

---

## 4. What the History Actually Looks Like

Using a real Tier 0 milestone with 8 tasks as the example.

**With --no-ff (what you want to avoid for solo work):**

```
Commits on Apr 15, 2026

Merge fix/race-condition into restructure/tier-0
fix: make orchestration task creation atomic
Merge fix/stale-task-recovery into restructure/tier-0
fix: add time threshold and error logging
Merge fix/hsts-security-headers into restructure/tier-0
fix: add HSTS and security headers
Merge chore/optional-fabric-deps into restructure/tier-0
chore: move pyodbc and pyarrow to optional
Merge fix/global-exception-handler into restructure/tier-0
fix: add global exception handler
Merge fix/cors-wildcard into restructure/tier-0
fix: replace CORS wildcard
Merge chore/pin-dbt-versions into restructure/tier-0
chore: pin dbt dependencies
Merge fix/dummy-credential into restructure/tier-0
fix: replace hardcoded Fabric server GUID
```

17 entries for 8 actual changes. Each task appears twice — once for the work commit and once for the merge event.

**With Squash (what you want):**

```
Commits on Apr 15, 2026

fix: make orchestration task creation atomic
fix: add time threshold and error logging
fix: add HSTS and security headers
chore: move pyodbc and pyarrow to optional
fix: add global exception handler
fix: replace CORS wildcard
chore: pin dbt dependencies
fix: replace hardcoded Fabric server GUID
```

8 entries for 8 actual changes. One line per task. Clean. Readable. Tells the story of what was done in this milestone at a glance.

---

## 5. Real World Scenario — What Happened and What Should Have Happened

**The situation:**
A developer took ownership of an inherited repository and worked through 8 security and stability fixes as part of Tier 0 restructuring. Each fix was done in a separate task branch and merged into a milestone branch `restructure/tier-0` using `--no-ff`.

**What they saw in GitHub:**
17 entries for 8 changes. The history showed every work commit and every merge commit interleaved — making it look like commits were duplicated when they were not.

**Why it happened:**
`--no-ff` explicitly creates a merge commit for every branch merge. This is correct behavior — it is doing exactly what it was told. The history was not broken, just noisier than expected.

**What should have been used:**
Squash and merge. Each task branch would have produced exactly one clean commit on the milestone branch regardless of how many commits were inside the task branch during development.

**The fix going forward:**
Create a fresh milestone branch `tier0/quick-wins` from main. Reimplement each task using squash merge. The original `restructure/tier-0` branch is preserved as is — nothing is lost.

---

## 6. The Milestone Branch Strategy

Understanding merge strategies only makes full sense in the context of how branches are organized.

**The three-level hierarchy:**

```
main
  └── milestone branch   (e.g. tier0/quick-wins, feature/phase-1)
        └── task branches (e.g. fix/cors-wildcard, feat/fix-loops)
```

**The flow:**

```
1. Create milestone branch from main
   git checkout main
   git checkout -b tier0/quick-wins

2. For each task:
   git checkout -b fix/cors-wildcard tier0/quick-wins
   do the work — commit freely, WIP commits are fine
   squash merge back into milestone branch
   delete task branch

3. When all tasks done:
   integration test everything on the milestone branch
   raise one PR: tier0/quick-wins → main
   one review, one approval, one clean merge to main
```

**Why the milestone branch matters:**

```
Main is always clean and deployable
    Nothing reaches main until an entire milestone is verified

Integration testing happens on the milestone branch
    All fixes are tested together before touching main
    A fix that works in isolation might break when combined with others
    The milestone branch is where you find that out safely

Manager or lead sees one PR per milestone
    Not 8 individual task PRs
    One coherent review: "Tier 0 complete, all tested, ready to merge"
```

---

## 7. Solo vs Team — When to Use What

**Solo developer:**

```
Squash merge locally — no PR per task needed

git checkout tier0/quick-wins
git merge --squash fix/cors-wildcard
git commit -m "fix: replace CORS wildcard with explicit origin"
git branch -d fix/cors-wildcard
git push origin tier0/quick-wins

GitHub never sees the task branch
GitHub only sees the clean milestone branch
One final PR from milestone to main
```

**Multiple engineers on the same milestone:**

```
Each engineer pushes their task branch to GitHub
Opens a PR into the milestone branch
Lead reviews and approves per task
Squash merge on GitHub
Auto-delete task branch after merge

Engineer A → fix/cors-wildcard → PR → tier0/quick-wins → squash merge
Engineer B → chore/pin-dbt    → PR → tier0/quick-wins → squash merge
Engineer C → fix/race-condition → PR → tier0/quick-wins → squash merge

Milestone branch history stays clean
One commit per task regardless of how many engineers or commits
```

The milestone branch is the integration point in both cases. The only difference is where the squash merge happens — locally for solo, on GitHub via PR for teams.

---

## 8. Squash and the Closed PR — Nothing Is Ever Lost

A common concern with squash is that the individual commits inside the task branch disappear. This is only partially true.

**What disappears:**
The individual task branch commits from the milestone branch history.

**What never disappears:**

```
The squash commit itself
    Lives on the milestone branch permanently
    Contains the clean commit message you wrote

The task branch
    Still exists in GitHub until you delete it
    All original commits fully browsable

The closed PR
    Permanent record in GitHub — Pull Requests → Closed tab
    Shows every commit that was in the task branch
    Shows the full diff, every file, every line
    Shows any review comments
    Cannot be deleted
    One click away forever
```

**When you need to go back and see what happened during a task:**

```
GitHub → Pull Requests → Closed tab
→ find the PR for fix/cors-wildcard
→ see every commit, every change, every line
→ full investigation trail intact
```

**Best practice for squash commit messages:**
Write a detailed body not just a title. This becomes the permanent record on the milestone branch.

```
fix: replace CORS wildcard with explicit origin

- Found that allow_origins=["*"] + allow_credentials=True violates
  the CORS spec — any website could make credentialed requests to the API
- Replaced wildcard with explicit CORS_ORIGINS environment variable
- Removed allow_credentials=True — not required for this use case
- Added CORS_ORIGINS to .env.example with safe default value
- Tested with request from disallowed origin — correctly rejected with 403

Ref: tier0/quick-wins — TIER-0.2
```

---

## 9. How to Set Squash as Default in GitHub

Set this once and every PR merge defaults to squash automatically.

```
Repo → Settings → General → Pull Requests section

Uncheck: Allow merge commits
Uncheck: Allow rebase merging
Check:   Allow squash merging        ← only this one

Set default commit message: Pull request title
```

Also enable auto-delete so task branches are cleaned up automatically after every merge:

```
Repo → Settings → General
→ Automatically delete head branches   ← enable this
```

Two settings. Set once. Clean history and clean branch list forever.

---

## 10. Quick Reference

**Which strategy produces what:**

```
--no-ff merge   → all commits + merge commit visible
                  noisiest, most history, clearest branch boundaries

Squash merge    → one clean commit, everything else hidden
                  cleanest, best for product teams and solo work

Rebase merge    → all commits visible, no merge commit
                  linear but WIP commits exposed

Fast Forward    → same as rebase, only possible in specific conditions
```

**Decision guide:**

```
Are you working solo?
    → Squash locally, push milestone branch, one PR to main

Are you on a team reviewing each task?
    → Push task branch, open PR to milestone, squash on GitHub

Do you need every individual commit visible forever?
    → Rebase (rare — usually only open source maintainers need this)

Do you never want WIP commits showing?
    → Squash always
```

**The commands for local squash merge:**

```bash
# Create task branch from milestone
git checkout -b fix/cors-wildcard tier0/quick-wins

# Do the work freely — WIP commits are fine here
git add .
git commit -m "wip: trying something"
git add .
git commit -m "fix: found the real issue"
git add .
git commit -m "fix: replace CORS wildcard with explicit origin"

# Squash merge into milestone
git checkout tier0/quick-wins
git merge --squash fix/cors-wildcard
git diff --cached --stat          # review what is staged
git commit -m "fix: replace CORS wildcard with explicit origin"

# Delete task branch locally
git branch -d fix/cors-wildcard

# Push clean milestone branch
git push origin tier0/quick-wins
```

**The git log command to verify clean history:**

```bash
git log --oneline tier0/quick-wins
```

Expected output — one line per task, no merge commits:

```
a1b2c3d fix: make orchestration task creation atomic
e4f5g6h fix: add time threshold and error logging
i7j8k9l fix: add HSTS and security headers
m1n2o3p chore: move pyodbc and pyarrow to optional
q4r5s6t fix: add global exception handler
u7v8w9x fix: replace CORS wildcard with explicit origin
y1z2a3b chore: pin dbt dependencies to exact versions
c4d5e6f fix: replace hardcoded Fabric server GUID
```

---

*Adrian Arante — Git Merge Strategies Reference*
*Engineering Standards Series*
