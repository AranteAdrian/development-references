# Managing Multiple GitHub Accounts on Windows

A practical guide for developers who need to switch between work and personal GitHub accounts seamlessly on a Windows machine.

---

## Table of Contents

1. [Overview](#overview)
2. [Prerequisites](#prerequisites)
3. [Generate Separate SSH Keys](#1-generate-separate-ssh-keys)
4. [Add SSH Keys to GitHub](#2-add-ssh-keys-to-github)
5. [Configure the SSH Config File](#3-configure-the-ssh-config-file)
6. [Clone Repos Using the Correct Host Alias](#4-clone-repos-using-the-correct-host-alias)
7. [Set Per-Repository Git Identity](#5-set-per-repository-git-identity)
8. [Automate Identity with Directory-Based Config](#6-automate-identity-with-directory-based-config)
9. [Quick Reference: Common Commands](#quick-reference-common-commands)
10. [Troubleshooting](#troubleshooting)

---

## Overview

When you use multiple GitHub accounts (e.g., work + personal) on the same machine, Git and SSH don't know which credentials to use by default. The solution is to:

- Create a **dedicated SSH key** for each account.
- Map each key to a **host alias** in your SSH config.
- Set the correct **Git user identity** per repository or directory.

This guide provides both **PowerShell** and **Git Bash** commands for Windows.

### The Journey at a Glance

```
 YOU ARE HERE
     │
     ▼
 ┌──────────────────────────────────────────────────────────────────────┐
 │  PREREQUISITES          Make sure your tools are ready              │
 └──────────────┬───────────────────────────────────────────────────────┘
                ▼
 ┌──────────────────────────────────────────────────────────────────────┐
 │  STEP 1: GENERATE KEYS   Create a unique "passport" per account    │
 └──────────────┬───────────────────────────────────────────────────────┘
                ▼
 ┌──────────────────────────────────────────────────────────────────────┐
 │  STEP 2: REGISTER KEYS   Hand each passport to the right GitHub    │
 └──────────────┬───────────────────────────────────────────────────────┘
                ▼
 ┌──────────────────────────────────────────────────────────────────────┐
 │  STEP 3: SSH CONFIG       Teach your machine which passport to use │
 └──────────────┬───────────────────────────────────────────────────────┘
                ▼
 ┌──────────────────────────────────────────────────────────────────────┐
 │  STEP 4: CLONE REPOS      Use the right identity when cloning      │
 └──────────────┬───────────────────────────────────────────────────────┘
                ▼
 ┌──────────────────────────────────────────────────────────────────────┐
 │  STEP 5: GIT IDENTITY     Make sure commits show the right author  │
 └──────────────┬───────────────────────────────────────────────────────┘
                ▼
 ┌──────────────────────────────────────────────────────────────────────┐
 │  STEP 6: AUTOMATE IT      Never think about switching again        │
 └──────────────────────────────────────────────────────────────────────┘
                ▼
            DONE — You now switch accounts effortlessly.
```

---

## Prerequisites

> **Purpose:** Before building anything, make sure the foundation is in place. You need Git and SSH available on your machine — everything else builds on these two tools.

- **Git for Windows** installed → [git-scm.com](https://git-scm.com/download/win)
  - This includes Git Bash and the `ssh-keygen` command.
- **OpenSSH** enabled (comes built-in on Windows 10/11).

To verify, open **PowerShell** and run:

```powershell
git --version
ssh -V
```

Both should return version info. If `ssh` is not recognized, enable it via **Settings → Apps → Optional Features → OpenSSH Client**.

---

## 1. Generate Separate SSH Keys

> **Purpose:** SSH keys are like digital passports — each one proves your identity to GitHub. Right now your machine only has your work "passport," so GitHub only recognizes you as your work account. In this step, you create a second passport for your personal account so both identities can coexist on the same machine.

> **Already have a work SSH key?** Many developers already have a key like `id_ed25519` or `id_rsa` from their initial work setup. If so, you only need to generate the **personal** key — no need to recreate the work one. You can check what you already have with `dir $env:USERPROFILE\.ssh\` in PowerShell.

Open **Git Bash** (recommended) or **PowerShell** and generate one key per account.

### Using Git Bash

```bash
# Generate key for your WORK account (skip if you already have one)
ssh-keygen -t ed25519 -C "you@work-email.com" -f ~/.ssh/id_ed25519_work

# Generate key for your PERSONAL account
ssh-keygen -t ed25519 -C "you@personal-email.com" -f ~/.ssh/id_ed25519_personal
```

### Using PowerShell

```powershell
# Generate key for your WORK account (skip if you already have one)
ssh-keygen -t ed25519 -C "you@work-email.com" -f "$env:USERPROFILE\.ssh\id_ed25519_work"

# Generate key for your PERSONAL account
ssh-keygen -t ed25519 -C "you@personal-email.com" -f "$env:USERPROFILE\.ssh\id_ed25519_personal"
```

When prompted for a passphrase, you can set one for added security or press Enter twice to skip.

> **What does the `-C` flag do?** The `-C "you@email.com"` part is just a **comment label** embedded in the key file — it helps you tell keys apart. It does **not** set your GitHub username, register your account, or affect authentication in any way. Your GitHub username is determined by which account you register the key with in Step 2.

**Verify the keys were created:**

```powershell
# PowerShell (use $env:USERPROFILE — the ~ shorthand can be unreliable in some commands)
dir $env:USERPROFILE\.ssh\

# Git Bash
ls -la ~/.ssh/
```

You should see four new files:

```
id_ed25519_work          (private key)
id_ed25519_work.pub      (public key)
id_ed25519_personal      (private key)
id_ed25519_personal.pub  (public key)
```

---

## 2. Add SSH Keys to GitHub

> **Purpose:** You've created the passports, but GitHub doesn't know about them yet. In this step, you register each public key with its matching GitHub account — like handing your passport to immigration so they can verify you at the gate.

Copy each public key and add it to the corresponding GitHub account.

### Using PowerShell

```powershell
# Copy WORK public key to clipboard
Get-Content $env:USERPROFILE\.ssh\id_ed25519_work.pub | Set-Clipboard

# Copy PERSONAL public key to clipboard
Get-Content $env:USERPROFILE\.ssh\id_ed25519_personal.pub | Set-Clipboard
```

### Using Git Bash

```bash
# Copy WORK public key to clipboard
clip < ~/.ssh/id_ed25519_work.pub

# Copy PERSONAL public key to clipboard
clip < ~/.ssh/id_ed25519_personal.pub
```

For each key, go to the **respective GitHub account**:

1. Navigate to **Settings → SSH and GPG Keys → New SSH Key**.
2. Give it a descriptive title (e.g., "Work Laptop" or "Personal - Work Laptop").
3. Key type: **Authentication Key**.
4. Paste the key and click **Add SSH Key**.

> **The title is just a label for you** — GitHub doesn't use it for authentication. Since these keys live on two different GitHub accounts, they'll never appear in the same list. Name them whatever helps you identify the machine later (e.g., "Dell Laptop", "Home Desktop").

> **You must complete this step before testing SSH connections.** If you skip registering the key on GitHub, `ssh -T github-personal` will fail with `Permission denied (publickey)` — not because your config is wrong, but because GitHub doesn't recognize the key yet.

---

## 3. Configure the SSH Config File

> **Purpose:** This is the brain of the whole setup. Your machine has two keys now, but it doesn't know which one to grab when you connect to GitHub. The SSH config file creates "aliases" (nicknames) — so when you say `github-work`, SSH automatically picks your work key, and when you say `github-personal`, it picks your personal key. Without this, SSH just guesses and usually picks wrong.

Create or edit the SSH config file to define a host alias for each account.

**The file location on Windows is:** `C:\Users\<YourUsername>\.ssh\config` (no file extension).

### Create the config file

#### Using PowerShell

```powershell
# Create the file if it doesn't exist
if (!(Test-Path ~\.ssh\config)) { New-Item -Path ~\.ssh\config -ItemType File }

# Open in Notepad
notepad ~\.ssh\config
```

#### Using Git Bash

```bash
nano ~/.ssh/config
```

### Add the following content

> **Adjust `IdentityFile` to match your actual key filenames.** If your work key is the default `id_ed25519` (not `id_ed25519_work`), use that name instead. Check with `dir $env:USERPROFILE\.ssh\` to see your exact filenames.

```text
# ── Work GitHub Account ──
Host github-work
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_work
    IdentitiesOnly yes

# ── Personal GitHub Account ──
Host github-personal
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_personal
    IdentitiesOnly yes
```

**Example if your work key is the default `id_ed25519`:**

```text
# ── Work GitHub Account ──
Host github-work
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519
    IdentitiesOnly yes
```

> **Note:** Use forward slashes (`/`) in the `IdentityFile` path even on Windows. SSH understands them.

**What each line does:**

| Directive        | Purpose                                                    |
| ---------------- | ---------------------------------------------------------- |
| `Host`           | Your custom alias — used in place of `github.com` in URLs  |
| `HostName`       | The actual server to connect to                            |
| `User`           | Always `git` for GitHub                                    |
| `IdentityFile`   | Points to the specific private key for this account        |
| `IdentitiesOnly` | Prevents SSH from trying other keys (avoids auth failures) |

### Start the SSH Agent and add your keys

#### PowerShell

> **The first two commands require Administrator privileges.** Right-click Start → **Terminal (Admin)** or search "PowerShell" → right-click → **Run as administrator**. The remaining commands do NOT need Admin.

**In Admin PowerShell (one-time setup):**

```powershell
# Command 1: Enable the SSH Agent service
Get-Service ssh-agent | Set-Service -StartupType Automatic

# Command 2: Start the service
Start-Service ssh-agent

# Command 3: Verify it's running (should show Status: Running)
Get-Service ssh-agent
```

**Back in your regular PowerShell (no Admin needed):**

```powershell
# Add both keys (use $env:USERPROFILE, NOT ~ — the tilde can fail with ssh-add)
ssh-add $env:USERPROFILE\.ssh\id_ed25519_work
ssh-add $env:USERPROFILE\.ssh\id_ed25519_personal

# Verify keys are loaded
ssh-add -l
```

> **PowerShell gotcha:** `ssh-add ~\.ssh\...` may fail with `No such file or directory` because `ssh-add` doesn't always expand `~` on Windows. Always use `$env:USERPROFILE` instead.

#### Git Bash

```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519_work
ssh-add ~/.ssh/id_ed25519_personal
```

### Test each connection

```bash
ssh -T github-work
# Expected: Hi <work-username>! You've been successfully authenticated...

ssh -T github-personal
# Expected: Hi <personal-username>! You've been successfully authenticated...
```

---

## 4. Clone Repos Using the Correct Host Alias

> **Purpose:** This is the payoff — you can now actually clone your personal repo. Instead of the standard `git@github.com:...` URL (which doesn't know which account to use), you swap in the alias from Step 3. This is the step that directly solves the "can't clone because I'm authenticated as work" problem.

Instead of the default `github.com`, use your alias when cloning.

### Understanding the clone URL

```
git clone git@github-personal:AranteAdrian/building-ai-agents-pure-python.git
               │                │              │
               │                │              └── Repository name (from GitHub)
               │                │
               │                └── Your GitHub USERNAME (from your GitHub account,
               │                    NOT from the SSH key or key generation)
               │
               └── SSH config alias (from Step 3 — tells SSH which key to use)
```

> **Common mistake:** Using `github.com` instead of your alias. If you clone with `git@github.com:...`, SSH will present whichever key it finds first (usually your work key). Your work account doesn't have access to your personal repos, so GitHub returns "Repository not found." Always use `github-personal` or `github-work` in place of `github.com`.

```bash
# Clone a WORK repo
git clone git@github-work:work-org/project-name.git

# Clone a PERSONAL repo
git clone git@github-personal:your-personal-username/repo-name.git
```

**For an existing repo**, update the remote URL to use the correct alias:

```powershell
cd C:\Users\aarante\your-existing-repo

# Check current remote
git remote -v

# Update to use the right alias
git remote set-url origin git@github-personal:your-personal-username/repo-name.git
```

**For a brand new repo** you just created on GitHub, the setup page will suggest an HTTPS URL like `https://github.com/...`. Do **not** use it. Use the SSH alias instead:

```bash
# GitHub suggests this — DO NOT USE:
git remote add origin https://github.com/AranteAdrian/my-new-repo.git

# Use this instead:
git remote add origin git@github-personal:AranteAdrian/my-new-repo.git
```

If you already added the HTTPS remote, fix it with:

```bash
git remote set-url origin git@github-personal:AranteAdrian/my-new-repo.git
```

> **Why HTTPS fails:** HTTPS authentication goes through Windows Credential Manager, which caches your work account login. SSH bypasses this entirely by using your key — which is why this whole guide uses SSH.

---

## 5. Set Per-Repository Git Identity

> **Purpose:** SSH handles *access* (who can push/pull), but Git handles *authorship* (whose name appears on commits). If you skip this step, your personal tutorial commits will still show your work name and email — which looks unprofessional and can leak your work identity to public repos. This step makes sure the right name is on the right commits.

Git uses `user.name` and `user.email` for every commit. Set the correct identity **inside each repo** so your commits are attributed to the right account.

```powershell
cd C:\Users\aarante\personal\my-tutorial

# Set identity for THIS repo only (no --global flag)
git config user.name "Your Personal Name"
git config user.email "you@personal-email.com"
```

**Verify:**

```bash
git config user.name
git config user.email
```

> **Important:** Avoid using `--global` when you have multiple accounts. Global config will bleed into repos where it doesn't belong.

---

## 6. Automate Identity with Directory-Based Config

> **Purpose:** Steps 1–5 work, but you'll have to manually set `user.name` and `user.email` every time you clone a new repo. This step eliminates that. By organizing repos into `work\` and `personal\` folders, Git automatically applies the correct identity based on where the repo lives. This is the "set it and forget it" step — once configured, you never think about it again.

Organize projects into directories and use Git's `includeIf` feature to auto-apply the right config.

### Recommended directory structure

```
C:\Users\aarante\
├── work\                ← All work repositories
│   ├── client-project\
│   └── internal-tool\
└── personal\            ← All personal repositories
    ├── tutorial-repo\
    └── side-project\
```

### Step 1 — Create separate Git config files

Create two new files in your home directory (`C:\Users\aarante\`):

**`.gitconfig-work`**

```ini
[user]
    name = Your Work Name
    email = you@work-email.com
```

**`.gitconfig-personal`**

```ini
[user]
    name = Your Personal Name
    email = you@personal-email.com
```

### Step 2 — Update your global `.gitconfig`

Open `C:\Users\aarante\.gitconfig` and add:

```ini
[includeIf "gitdir:C:/Users/aarante/work/"]
    path = .gitconfig-work

[includeIf "gitdir:C:/Users/aarante/personal/"]
    path = .gitconfig-personal
```

> **Windows-specific notes:**
> - Use **forward slashes** (`/`) in the `gitdir` path, not backslashes.
> - The **trailing slash** after the directory name is required.
> - Paths are **case-insensitive** on Windows.

Now any repo under `work\` automatically uses your work identity, and anything under `personal\` uses your personal identity.

---

## Quick Reference: Common Commands

> **Purpose:** A cheat sheet for day-to-day use. Bookmark this section — once the setup is done, these are the only commands you'll need.

### PowerShell

```powershell
# ─── Test SSH connections ───
ssh -T github-work
ssh -T github-personal

# ─── Clone with specific account ───
git clone git@github-work:org/repo.git
git clone git@github-personal:user/repo.git

# ─── Switch remote on existing repo ───
git remote set-url origin git@github-personal:user/repo.git

# ─── Check current repo identity ───
git config user.name
git config user.email

# ─── Check which SSH key is being used ───
ssh -vT github-personal 2>&1 | Select-String "Offering public key"

# ─── List loaded SSH keys ───
ssh-add -l

# ─── Manually add a key to SSH agent ───
ssh-add $env:USERPROFILE\.ssh\id_ed25519_personal

# ─── Remove cached GitHub credentials (if using HTTPS) ───
# Open: Control Panel → Credential Manager → Windows Credentials
# Remove any entries for github.com
```

### Git Bash

```bash
# ─── Test SSH connections ───
ssh -T github-work
ssh -T github-personal

# ─── Clone with specific account ───
git clone git@github-work:org/repo.git
git clone git@github-personal:user/repo.git

# ─── Check which SSH key is being used ───
ssh -vT github-personal 2>&1 | grep "Offering public key"

# ─── List loaded SSH keys ───
ssh-add -l
```

---

## Troubleshooting

> **Purpose:** Things don't always work on the first try. This section covers the most common issues and how to fix them fast.

### "Permission denied (publickey)" when cloning or testing

This is the most common error. Check these in order — the first one catches most cases:

1. **Did you register the public key on GitHub?** This is the #1 cause. Go to GitHub → Settings → SSH and GPG Keys and confirm the key is listed on the correct account. If not, go back to Step 2.
2. Confirm you're using the **alias** (e.g., `github-personal`), not `github.com`.
3. Test with `ssh -T github-personal` to verify authentication.
4. Make sure the SSH key is added to the agent: `ssh-add $env:USERPROFILE\.ssh\id_ed25519_personal`.
5. Double-check that the `IdentityFile` path in `~/.ssh/config` matches your actual key filename.

### "Repository not found" when cloning your own repo

This usually means SSH sent the **wrong key**, and GitHub authenticated you as a different account that doesn't have access to the repo.

```
# WRONG — This uses your DEFAULT key (probably work) — GitHub sees your work account
git clone git@github.com:YourPersonalUser/your-repo.git

# RIGHT — This uses your PERSONAL key via the alias
git clone git@github-personal:YourPersonalUser/your-repo.git
```

The fix is always: replace `github.com` with the correct alias (`github-personal` or `github-work`). If the repo is yours and you can see it in your browser, the issue is the key, not the repo.

### SSH Agent not running / "Error connecting to agent"

If you see `Error connecting to agent: No such file or directory` when running `ssh-add`, the SSH Agent service isn't started yet.

**Fix — open PowerShell as Administrator** (right-click Start → Terminal (Admin)):

```powershell
# Check the current state
Get-Service ssh-agent

# If Status is "Stopped" or "Disabled", run:
Get-Service ssh-agent | Set-Service -StartupType Automatic
Start-Service ssh-agent
```

> If you get `Access is denied`, you are **not** running as Administrator. Close your terminal and reopen it with right-click → **Run as administrator**.

Once the service is running, go back to your **regular (non-admin)** PowerShell and retry `ssh-add`.

### "No such file or directory" when running `ssh-add`

If `ssh-add ~\.ssh\id_ed25519_personal` fails but the file clearly exists, the issue is that `ssh-add` on Windows doesn't always expand the `~` (tilde) shorthand.

**Fix — use the full path via `$env:USERPROFILE`:**

```powershell
# This may FAIL:
ssh-add ~\.ssh\id_ed25519_personal

# This will WORK:
ssh-add $env:USERPROFILE\.ssh\id_ed25519_personal
```

Also make sure you've actually generated the key first (Step 1) before trying to add it. Run `dir $env:USERPROFILE\.ssh\` to confirm the file exists.

### Commits showing the wrong author

Run `git log --format="%an <%ae>"` to check. If the author is wrong, fix with:

```bash
git config user.name "Correct Name"
git config user.email "correct@email.com"
```

To rewrite the last commit's author (before pushing):

```bash
git commit --amend --reset-author --no-edit
```

### Still authenticated as the wrong account (HTTPS users)

If you previously cloned repos via HTTPS, Windows may have cached credentials in **Credential Manager**.

1. Press `Win + S` and search for **Credential Manager**.
2. Click **Windows Credentials**.
3. Find entries for `git:https://github.com`.
4. Click the entry and select **Remove**.
5. Switch to SSH-based remotes using the alias approach in this guide.

### Git Bash vs PowerShell — which should I use?

Either works. **Git Bash** behaves like a Linux terminal (uses `~`, `ls`, `cat`). **PowerShell** is native to Windows (uses `$env:USERPROFILE`, `dir`, `Get-Content`). This guide provides commands for both — pick whichever you're more comfortable with and stay consistent.

---

## Summary

| Step | What You Do                        | Why                                  |
| ---- | ---------------------------------- | ------------------------------------ |
| 1    | Generate separate SSH keys         | Each account gets its own credential |
| 2    | Add public keys to GitHub          | GitHub recognizes each key           |
| 3    | Configure `~/.ssh/config`          | Map aliases to the right keys        |
| 4    | Clone using the alias              | SSH picks the correct key            |
| 5    | Set repo-level Git identity        | Commits show the right author        |
| 6    | Use `includeIf` for auto-identity  | No more manual config per repo       |

Once set up, switching accounts is effortless — just clone or work inside the right directory and everything routes automatically.
