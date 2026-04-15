# Project Reusable Instruction Patterns

> These are reusable instruction prompts designed for consistent, structured AI-assisted engineering work. Each pattern is a ready-to-use template with placeholders you fill in per project. Copy, replace the placeholders, and paste into Claude Code or Claude chat whenever the scenario applies.

---

## 1. Let Claude Code Scan Through Your Plan MD for GitHub Tickets

**Use it when:**
- Your audit is complete, your plan is documented in MD, tickets are created in GitHub, and you want the AI to read the plan and work through it without manual handholding per issue
- Instead of manually opening each GitHub ticket and copying it into Claude Code or GitHub Copilot one by one, let Claude Code scan your plan MD file and drive execution ticket by ticket

---

```
I have taken ownership of this repository and will be performing structured
housekeeping, cleanup, and refactoring based on a full pre-audit I completed.

**What has already been done:**
The audit covers business context, codebase assessment, feature mapping,
UI mapping, and production-readiness opportunities. All findings are
documented in [AUDIT_FOLDER].

**Primary task source:**
Use [TASK_SOURCE_FILE] as the master list of tasks to work through.

**Additional references:**
All audit findings in the [AUDIT_FOLDER] folder are available as supporting
context for any implementation decision.

**Starting point:**
Begin with the Restructuring Roadmap, [STARTING_TIER_OR_PHASE] tasks only.

**Branch naming standard:**
Create a dedicated branch per issue following this convention:
fix/cors-wildcard
fix/race-condition
chore/pin-dbt-versions
chore/docker-compose
refactor/extract-worker-process
feat/fix-loops
feat/wave-runner

**Workflow:**
Work through [STARTING_TIER_OR_PHASE] issues one at a time.
After completing each branch, stop and wait for my approval before
proceeding to the next issue.
Do not move to the next tier or phase until all current items are approved.
```
