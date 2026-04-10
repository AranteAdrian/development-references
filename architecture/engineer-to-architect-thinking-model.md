# Engineer to Architect — Thinking Model & Growth Reference

A scalable collection of comparisons, thinking patterns, and lessons learned on the path from engineer to architect. Add new scenarios as you encounter them.

---

## Scenario 1: Repo Takeover — Engineer vs Architect

**The Situation:** You're handed a repo you've never seen. The previous team is gone. Docs are outdated. Stakeholders are asking "when can you ship the next feature?"

### How an Engineer Responds

**The pattern:** Jump into code → learn reactively → fix what's in front of them → discover context gaps the hard way.

Not wrong — but the understanding is fragmented, slow, and shaped by whatever fires they encounter first.

### How an Architect Responds

**The pattern:** Understand before acting → map the system → assess strategically → then execute with purpose.

### The Core Differences

| Dimension | Engineer | Architect |
|---|---|---|
| **First question** | "How do I make this work?" | "Why does this exist and who depends on it?" |
| **How they learn the system** | Bottom-up: read code, trace execution | Top-down: business context → architecture → code |
| **What they fix first** | Whatever's annoying or broken | Whatever has the highest impact on the system's health |
| **How they handle stale docs** | Trust them, get burned, then ignore all docs | Verify against code, flag contradictions, produce new docs |
| **How they communicate findings** | "This code is bad" | "This pattern doesn't fit because the workload changed — here's what it should be and why" |
| **How they prioritize** | By visibility (what's complained about) | By dependency (fix foundations before walls) |
| **What they produce** | Working code | Working code + the context for everyone who comes after them |
| **Time horizon** | "Does it work now?" | "Will this still work in 2 years when traffic is 10x?" |
| **Response to "just ship it"** | Ships it | Explains the risk of shipping without addressing X, then proposes a path that ships AND addresses X |

### The One Line That Separates Them

> An engineer changes the system. An architect changes how the team understands the system — and then changes it in the right order.

---

## Self-Assessment: Am I Thinking at an Architect Level?

### Honest Breakdown

| Architect Skill | Evidence | Level |
|---|---|---|
| **Problem definition before solution** | Started from the human pain, not the tool | Architect |
| **Audience awareness** | Different audiences per step, caught wrong audience in Step 3 | Architect |
| **Systems thinking** | Progressive context chain, dependency ordering | Architect |
| **Reusable design** | Extracted shared rules, built a framework not a one-off | Architect |
| **Trade-off reasoning** | Current state vs target, alternatives vs chosen pattern | Architect |
| **Failure mode identification** | Stale docs, blind fixing, missing context | Senior → Architect |
| **Technical breadth** | Asked for alternatives to expand own knowledge — self-aware gap | Senior, actively closing the gap |
| **Implementation depth** | Knows architecture patterns but wants to deepen hands-on experience | Senior, building reps |

### The Gap

Technical breadth is still growing. That's not a weakness — it's normal. Architects don't know everything; they know **how to evaluate options they haven't used before**. Building a prompt rule that forces alternatives to be listed is closing that gap systematically.

### What Separates Me from a Titled Architect Right Now

It's not the thinking — it's the **reps on real systems**. I need more cycles of:

1. Making architectural decisions on production systems
2. Living with the consequences
3. Adjusting based on what actually happened vs. what I predicted

The thinking model is there. The process is there. The instinct is there. Now it needs mileage.

---

<!-- 
=== HOW TO ADD NEW SCENARIOS ===

Copy and paste this template below, fill in the details:

## Scenario N: [Title] — Engineer vs Architect

**The Situation:** [Describe the real-world scenario]

### How an Engineer Responds

**The pattern:** [Describe the typical engineer approach]

### How an Architect Responds

**The pattern:** [Describe the architect approach]

### The Core Differences

| Dimension | Engineer | Architect |
|---|---|---|
| **[Dimension]** | [Engineer response] | [Architect response] |

### Key Takeaway

> [One line summary]

-->
