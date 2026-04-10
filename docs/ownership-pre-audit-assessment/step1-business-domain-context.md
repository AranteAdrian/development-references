# 360° Repo Takeover — Part 1: Business Context & Domain

## WHY THIS PROMPT EXISTS

When a team takes over a project, the most common failure is not technical — it is contextual. Engineers inherit code without understanding what business problem it solves, who uses it, or why it was built this way. They ask scattered questions, get incomplete tribal knowledge from people who are already moving on, and spend weeks piecing together context that should have been documented on day one. Nobody talks about this — but it silently drains productivity every time a project changes hands. This prompt exists to eliminate that gap — to produce the business context document that should have existed from the start, so that anyone joining the project can go from zero to fully informed in a single read.

---

You are a software architect in the top 1% of your field. Your greatest strength is not just knowing architecture deeply — it is your ability to explain complex systems in simple, clear language that any audience can understand, from junior developers to non-technical stakeholders. You are performing Part 1 of a 360° Repo Takeover — the Business Context & Domain audit.
Your audience includes junior developers, business analysts, stakeholders, and anyone joining this project who has ZERO context about the business problem being solved, the domain this app operates in, or the pain points it addresses.

## GOAL

By the end of this document, the junior developer should be able to:

1. Explain the business problem and domain to any stakeholder.
2. Walk through every feature and how it maps to the business need.
3. Explain how this solution transforms the business's traditional workflow and what long-term value it delivers.
4. Trace the full pipeline or workflow end-to-end and know which ecosystem tools each step touches.
5. Defend why this app needs to exist when someone asks "why not just use X?"

Make the business context as clear as glass. Produce the following sections in order. Each section builds on the previous one.

---

## Section 1: Before You Read Anything Else — The World This App Lives In

Write this section for someone who has just joined the project and knows nothing about the domain. They should read this first. Everything else in the audit will make sense once they understand the problem being solved and the ecosystem the app operates in.

### 1a. The Problem in Plain English

Start from the widest possible angle. Don't jump straight to what the app does — first explain the world the app lives in so a junior developer understands the full context.

Follow this funnel:

1. **The industry/domain context** — What does the business world look like in this space? What do companies do, and how do they typically do it? Paint the picture of the "normal" way things work today, even if the app's users already know this. The junior developer might not.

2. **How it traditionally works** — Explain the traditional approach, tools, or workflows that companies use to solve this kind of problem. Use concrete examples that make the reader see the day-to-day reality. Then explain why that approach has worked but is now breaking down — list each pain point vividly so the reader feels the weight of the problem. Here is the level of depth and storytelling expected:

   > *"Traditionally, companies build these analytics using stored procedures — blocks of SQL code that live inside the database itself. A stored procedure might read from five raw tables, join them together, apply business rules (discount calculations, tax logic, date filtering), and write the result into a summary table. That summary table then feeds a dashboard or report that a manager looks at every morning.*
   >
   > *This approach has worked for decades. But it has serious problems:*
   >
   > *Nobody knows what the stored procedures do. They were written years ago by someone who left the company. The logic is undocumented. There are 200 of them, and they depend on each other in ways nobody fully understands.*
   >
   > *They are impossible to test. There is no way to verify that a stored procedure produces correct results other than manually spot-checking numbers in a spreadsheet.*
   >
   > *They are locked to one database. If the company wants to move from SQL Server to a modern cloud platform (like Microsoft Fabric), every stored procedure must be manually rewritten. This takes months of consultant time and costs hundreds of thousands of dollars.*
   >
   > *Mistakes are discovered too late. After migration, someone notices that the revenue report shows different numbers. But by then the old system has been decommissioned. Tracking down the discrepancy is a nightmare."*

   Adapt this same storytelling depth and specificity to whatever domain and problem the codebase you are auditing operates in.

3. **Why the traditional approach breaks down** — List the specific pain points. Be vivid and specific. Not "it's hard to maintain" but "Nobody knows what the stored procedures do. They were written years ago by someone who left the company. The logic is undocumented. There are 200 of them, and they depend on each other in ways nobody fully understands." Each pain point should make the junior developer feel the weight of the problem.

4. **Why it's expensive / risky / unsustainable** — Put numbers or time estimates on the pain where possible. "This takes months of consultant time and costs hundreds of thousands of dollars." Make the business case visceral.

5. **What this app does about it** — Only now, state what the app solves. One bold sentence that ties everything together.

The goal: a junior developer who reads this section should understand not just WHAT the app does, but WHY it needs to exist — and be able to explain that to a non-technical stakeholder.

### 1b. The Ecosystem — What Each Piece Does

List every major tool, platform, technology, or concept a junior developer needs to understand to work in this codebase. For each one:

- What it is
- What role it plays in this app specifically
- How it fits into the bigger picture

Use the same format as explaining tools to someone who has never heard of them. If the app uses dbt, explain dbt. If it uses Kafka, explain Kafka. If it uses a domain-specific concept like "lineage" or "semantic model," define it here.

### 1c. Why This App Exists — The Full Picture

Show the before/after contrast:

- What does a person do today WITHOUT this app? (step by step, with time estimates)
- What happens WITH this app? (step by step)
- State the core value proposition.

### 1d. Business Model — How This App Is Used in Practice

Explain:

- Who buys/uses this and why? (personas)
- Is this a permanent platform, a migration tool, a developer tool, an internal tool?
- What deliverables or outputs does the app produce?
- What this app is NOT (clarify common misconceptions about its scope)

Then explain **how this solution transforms the business's traditional way of operating:**

- **Before vs. After from the business user's perspective** — Show that the end user (the non-technical person at the end of the chain) sees the same output. The transformation is invisible to them. Explain what changed under the hood and why it matters to the technical team.
- **What the engagement or adoption looks like** — Who runs the app, who receives the output, and how does it get handed off? Is this something a client installs, or something a team runs on behalf of a client? Show the flow between the team running the app and the people receiving the deliverables.
- **Where the app stays relevant beyond the initial use** — Are there scenarios where the app is used more than once? Does it stay deployed, or is it decommissioned after the job is done?
- **The long-term value** — What permanent improvement does the business gain even after this app is no longer running? (e.g., "their analytics logic is never locked to a single database vendor again")

### 1e. Glossary — Terms You Will See Everywhere in This Codebase

Build a table of every domain term, acronym, and piece of jargon that appears in the codebase. For each term, write a plain-English definition a junior developer can understand on first read.

| Term | What it means |
|------|---------------|
| ... | ... |

### 1f. How the Core Pipeline / Workflow Maps to This Ecosystem

Now that the junior developer knows the tools, concepts, and glossary — connect the dots. Walk through the app's main pipeline or workflow step by step and explain what each step actually does using the ecosystem concepts from 1b.

For each step:

- What phase or step name it has in the codebase
- What it does in plain English
- Which ecosystem tools/concepts it touches
- How it feeds into the next step

If the app has an iteration or self-correcting loop, call it out explicitly — explain what triggers it, what it tries to fix, and when it stops.

This section is the bridge between "I understand the world this app lives in" and "I understand how the code is organized."

### 1g. Common Misconceptions — Questions You Will Ask (And the Answers)

After reading the business context, ecosystem, and pipeline, a junior developer (or a stakeholder) will inevitably push back with obvious questions like:

- "Why not just use [existing tool/feature] to solve this?"
- "Doesn't [technology X] already do this?"
- "Why build this when [simpler alternative] exists?"

Anticipate those questions and answer each one thoroughly. For each misconception:

- State the question as someone would naturally ask it
- Explain what the existing tool/feature actually does vs. what people assume it does
- Show the gap between what that tool covers and what this app covers — use concrete comparisons (tables, time estimates, percentage breakdowns) to make the gap visceral
- Use an analogy to make it stick (e.g., "The adapter is Google Translate. This app is the bilingual lawyer.")
- End with a clear statement of what this means for the product's value proposition

This section prevents the junior developer from spending weeks wondering "why does this app even exist when [X] is right there?" — a question that every new team member will have.

---

## Section 2: What This App Does (Technical Summary)

Write a technical summary of the entire application: what it takes as input, what it produces as output, and what drives the process in between. This is the technical version of Section 1.

---

## Section 3: Solution & Feature Audit

Now that we know the problem, what does the app actually DO?

- List every major feature/module the app has.
- For each feature, explain what it does and why it exists (tie it back to the business domain from Section 1).
- Map the primary user flow end-to-end (e.g., User signs up → creates X → triggers Y → receives Z).
- Does the main flow fully represent the core business use case from Section 1? Flag any gaps or half-built features.
- Identify any secondary/supporting flows (auth, notifications, admin panel, reporting, etc.)

---

## OUTPUT FORMAT

Create a folder named `ownership-pre-audit-findings/` in the root of the repository. This folder is for documentation purposes only — it contains the audit artifacts generated before any refactoring begins.

Save the output as `ownership-pre-audit-findings/BUSINESS_CONTEXT.md`.

For each section, use this structure:

```
## Section [N]: [Name]
**Summary:** (executive summary)
**Findings:** (detailed answers to each question above)
```

Write everything in plain language that anyone — developers, business analysts, stakeholders — can understand. Define jargon on first use. Avoid assumptions.