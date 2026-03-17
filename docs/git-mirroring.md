# Git Mirror Repository Guide

A step-by-step reference for mirroring someone else's repository into your own GitHub account, setting up remotes, and staying in sync.

---

## When to Use Mirroring

**Following along with a tutorial or course repo** — You want your own copy of an instructor's repository so you can write code, make commits, and push your progress to your own GitHub without affecting the original.

**Using someone's project as a starting point** — You found an open-source template, boilerplate, or reference architecture that you want to build on top of. Mirroring gives you a clean independent copy with full history, no "forked from" label on GitHub.

**Preserving a repo you depend on** — A library or tool you rely on might get deleted, archived, or go private. Mirroring it to your own account gives you a backup with all branches and tags intact.

**Moving a repo between GitHub accounts or orgs** — You're transferring a project from a personal account to a work org (or vice versa) and want to bring over everything including all branches, tags, and commit history.

**Creating a team reference copy** — You want your engineering team to have their own copy of an external repo (e.g., best practices, cookbooks, SDKs) that they can annotate, extend, or adapt without depending on the original owner's availability.

**Forking is not an option** — Some workflows or org policies don't allow GitHub forks, or you want a repo that is fully detached from the original with no fork relationship visible on GitHub.

---

## Prerequisites

Before starting, make sure you have:

- Git installed on your system
- A GitHub account with SSH or HTTPS access configured
- Created a **new empty repository** on your GitHub (no README, no .gitignore, no license)

---

## Step 1: Bare Clone the Original Repository

This creates a temporary copy of the original repo with all branches and history.

```bash
git clone --bare https://github.com/ORIGINAL-OWNER/ORIGINAL-REPO.git
```

This will create a folder called `ORIGINAL-REPO.git`.

---

## Step 2: Mirror-Push to Your New Repository

Push everything (all branches, tags, history) to your own empty repo.

```bash
cd ORIGINAL-REPO.git
git push --mirror https://github.com/YOUR-USER/YOUR-REPO.git
```

> If you use SSH, replace the URL accordingly:
> `git push --mirror git@github.com:YOUR-USER/YOUR-REPO.git`

---

## Step 3: Remove the Bare Clone

The bare clone is not usable for development. Delete it.

```bash
cd ..
rm -rf ORIGINAL-REPO.git
```

> **Windows PowerShell:** `Remove-Item -Recurse -Force ORIGINAL-REPO.git`

---

## Step 4: Clone Your Own Repository

Now clone your own repo normally so you can work on it locally.

```bash
git clone https://github.com/YOUR-USER/YOUR-REPO.git
cd YOUR-REPO
```

At this point, you have one remote:

| Remote   | Points To       |
|----------|-----------------|
| `origin` | Your own repo   |

---

## Step 5: Add Upstream Remote

Link the original repository so you can pull future updates from it.

```bash
git remote add upstream https://github.com/ORIGINAL-OWNER/ORIGINAL-REPO.git
```

Verify both remotes are set up:

```bash
git remote -v
```

Expected output:

```
origin    https://github.com/YOUR-USER/YOUR-REPO.git (fetch)
origin    https://github.com/YOUR-USER/YOUR-REPO.git (push)
upstream  https://github.com/ORIGINAL-OWNER/ORIGINAL-REPO.git (fetch)
upstream  https://github.com/ORIGINAL-OWNER/ORIGINAL-REPO.git (push)
```

| Remote     | Points To             | Purpose                        |
|------------|-----------------------|--------------------------------|
| `origin`   | Your own repo         | Where you push your work       |
| `upstream` | Original owner's repo | Where you pull their updates   |

---

## Step 6: Fetch Updates from Upstream

When the original repo has new commits you want to pull in:

```bash
git fetch upstream
git merge upstream/<branch-name>
```

Then push the merged changes to your own repo:

```bash
git push origin <branch-name>
```

> **Important:** Replace `<branch-name>` with the actual default branch. Check with `git branch -a` — it could be `main` or `master` depending on the repo.

---

## Checking Branches

| Command              | What It Shows                              |
|----------------------|--------------------------------------------|
| `git branch`         | Local branches only                        |
| `git branch -a`      | All branches (local + remote)              |
| `git remote -v`      | All remotes with their URLs                |
| `git remote get-url upstream` | Just the upstream URL             |

---

## Quick Reference: Full Workflow Summary

```bash
# 1. Bare clone
git clone --bare https://github.com/ORIGINAL-OWNER/ORIGINAL-REPO.git

# 2. Mirror-push to your repo
cd ORIGINAL-REPO.git
git push --mirror https://github.com/YOUR-USER/YOUR-REPO.git

# 3. Cleanup bare clone
cd ..
rm -rf ORIGINAL-REPO.git

# 4. Clone your repo for local work
git clone https://github.com/YOUR-USER/YOUR-REPO.git
cd YOUR-REPO

# 5. Add upstream
git remote add upstream https://github.com/ORIGINAL-OWNER/ORIGINAL-REPO.git

# 6. Verify remotes
git remote -v

# 7. Sync updates (whenever needed)
git fetch upstream
git merge upstream/<branch-name>
git push origin <branch-name>
```

---

## Notes

- **main vs master:** The default branch name depends on when and how the repo was created. `master` was the old default; `main` is the new default since 2020. Always check with `git branch -a`.
- **This is not a fork.** Your repo is fully independent. GitHub won't show a "forked from" link.
- **Upstream is read-only for you.** You fetch from upstream but never push to it (you don't have write access to someone else's repo).

---

## Additional Resources

- [GitHub Docs: Duplicating a Repository](https://docs.github.com/en/repositories/creating-and-managing-repositories/duplicating-a-repository)