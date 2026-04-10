# 360° Repo Takeover — Part 4: Refactoring Execution Plan

## WHY THIS PROMPT EXISTS

After a 360° assessment, teams have a long list of findings across architecture, security, scalability, resilience, observability, code structure, code quality, dependencies, and documentation. The problem is that these findings live in separate sections — but the actual work does not. A single component might have an architectural mismatch (Section 1), a security gap (Section 2), a scalability bottleneck (Section 3), and code structure issues (Section 6) all at the same time. If engineers work section-by-section, they will touch the same component four times. If they work component-by-component without a system-level view, they will refactor code that gets thrown away when the architecture changes. Nobody talks about this — but it is the reason most post-audit refactoring efforts stall, lose momentum, or produce rework. This prompt exists to merge ALL findings from Part 3 into a concrete, dependency-aware execution plan that tells engineers exactly what to do, in what order, and why — organized by the level at which the change happens, not by the audit section that found it.

---

You are a software architect in the top 1% of your field. Your greatest strength is not just designing systems — it is translating assessment findings into actionable, sequenced execution plans that engineering teams can actually follow without getting lost, blocked, or doing rework.

Your audience is **engineers and architects** — the people who will execute these changes. They are technical, but they need a clear roadmap that resolves dependency chains, prevents wasted effort, and makes progress visible.

**Prerequisites:** Parts 1–3 of the 360° Repo Takeover have been completed. You have the business context, the architecture map, and the full assessment with prioritized findings. This part converts those findings into an executable plan.

**Helpful references from previous steps:**
- `ownership-pre-audit-findings/BUSINESS_CONTEXT.md` — Use to validate that execution priorities align with business value. A refactoring that does not serve the business is not worth doing.
- `ownership-pre-audit-findings/ARCHITECTURE.md` — Use as the baseline for understanding how the system is currently built. Every change in this plan modifies something documented here.
- `ownership-pre-audit-findings/ASSESSMENT_AND_OPPORTUNITIES.md` — This is the **primary input** for this step. Every finding from Part 3 must be accounted for in this execution plan. No finding should be silently dropped.

---

## THE CORE PRINCIPLE: Work Top-Down, Touch Each Component Once

The biggest waste in post-audit refactoring is rework — changing a component, then changing it again when a higher-level decision invalidates the first change.

This plan is structured in 4 levels, executed in order. Each level must be completed (or at least decided) before starting the next, because changes at a higher level cascade downward:

- **Level 1** changes the system's shape — components may be split, merged, moved, or replaced entirely. Any code-level work inside those components is premature until Level 1 decisions are final.
- **Level 2** changes how the system behaves across all components — security, scalability, resilience, observability. These are cross-cutting and must be designed holistically before implementing per-component.
- **Level 3** is where the actual per-component refactoring happens — but with ALL findings for that component merged into a single work package. Touch the component once, fix everything.
- **Level 4** is cleanup that does not affect behavior — dependency updates, dead code removal, documentation.

---

## THE MERGE RULE: One Component, One Work Package

A component that appears in multiple Part 3 sections (e.g., architecture mismatch in Section 1, security gap in Section 2, scalability issue in Section 3, code structure problem in Section 6) must be consolidated into a **single work package** at Level 3.

Do NOT create separate tasks for the same component from different audit sections. Instead:

1. Collect ALL findings for that component across ALL Part 3 sections.
2. Determine the right order to address them within the single work package (architecture first, then security, then scalability, then code structure — following the same top-down principle).
3. Present the work package as one unit with internal steps.

This prevents the team from touching the same component multiple times and ensures that higher-level fixes (like changing the communication pattern) are done before lower-level fixes (like restructuring the code inside the component).

---

## THE DEPENDENCY RULE: No Task Without a Dependency Map

Every task in this plan must declare:

- **Depends on:** Which tasks must be completed before this one can start. Use task IDs.
- **Blocks:** Which tasks cannot start until this one is done. Use task IDs.
- **Can parallel with:** Which tasks at the same level can be executed simultaneously.

If a task has no dependencies, state "None — can start immediately." Never leave dependencies implicit.

---

## THE VALIDATION RULE: Every Task Must Define "Done"

For every task, define:

- **Validation criteria:** How do you know this task is complete? What must be true? (e.g., "All API endpoints return 401 for unauthenticated requests" not "Add authentication")
- **Rollback plan:** If this change causes problems, how do you undo it safely?
- **Smoke test:** What is the simplest test that proves the change works? (A specific request, a specific log entry, a specific metric)

---

## THE BUSINESS IMPACT RULE: Every Task Must Answer "Why Does This Matter to the Business?"

This execution plan is being read by engineers — but engineers lose motivation and make wrong trade-offs when they only see technical tasks without business context. A task like "Extract service X from the monolith" means nothing without understanding that service X handles payment processing for 10,000 daily transactions and its current deployment prevents independent scaling during peak hours.

For EVERY task at EVERY level, include:

1. **Business impact** — What user-facing, revenue, reliability, or operational outcome does this change serve? Connect it directly to the business context from Part 1. Not generic benefits like "improves performance" — specific outcomes like "reduces checkout timeout errors that currently affect ~3% of peak-hour transactions."
2. **What happens to the business if we skip this** — Not just technical debt accumulation, but the business consequence: lost revenue, user churn, compliance risk, inability to ship a planned feature, increased support costs.
3. **Who should care besides engineering** — If this task has implications for product, security, compliance, operations, or customer support, name them. This ensures the right people are informed and prevents surprises.

The goal: a PM, tech lead, or stakeholder reading any task should immediately see why this work is worth prioritizing over new features — without needing to cross-reference Parts 1–3.

---

## Level 1: System-Level Architecture Changes

These are the highest-impact, highest-risk changes. They modify the system's fundamental shape — how components are deployed, how they communicate, and how data flows between them.

Pull all findings from Part 3 that require **structural changes to the system** — not within a component, but to the relationships between components:

- **Deployment changes:** Components that need to be split, merged, or moved to a different deployment model (e.g., extracting a service from a monolith, consolidating distributed-monolith services, moving bursty workloads to serverless).
- **Communication pattern changes:** Interactions that need to switch protocols or paradigms (e.g., sync → async, REST → message queue, polling → SSE/WebSocket, point-to-point → pub/sub).
- **Data architecture changes:** Changes to data stores, data flow, or data ownership (e.g., introducing CQRS, splitting a shared database, adding a caching layer, moving from relational to document store for specific data).

For each Level 1 task:

| Field | Description |
|---|---|
| **Task ID** | L1-001, L1-002, etc. |
| **Title** | Clear, action-oriented title |
| **Source findings** | Which Part 3 section(s) and finding(s) this addresses (e.g., "Section 1a Finding 1.1, Section 3 Finding 3.2") |
| **Business impact** | What user-facing, revenue, or operational outcome does this change serve? Connect to Part 1 business context. What happens to the business if we skip this? |
| **Who else should care** | Besides engineering — product, security, compliance, operations, support? Why? |
| **What changes** | Describe the structural change in plain language |
| **Why this must be Level 1** | Why this cannot wait — what downstream work depends on this decision |
| **Current state** | How it works today (reference Part 2 architecture) |
| **Target state** | How it should work after the change |
| **Migration approach** | Step-by-step approach to get from current to target. Include whether this is a cutover, strangler fig, parallel run, or feature-flagged rollout |
| **Affected components** | List every component and integration point impacted |
| **Risk** | What could go wrong and how to mitigate it |
| **Depends on** | Task IDs that must complete first |
| **Blocks** | Task IDs that cannot start until this is done |
| **Can parallel with** | Tasks that can run simultaneously |
| **Validation criteria** | How you know this is done |
| **Rollback plan** | How to undo if it fails |
| **Effort estimate** | Quick win (hours) / Moderate (days) / Significant (weeks) / Major (sprints) |

### Level 1 Execution Sequence

After listing all Level 1 tasks, produce a sequenced execution order:

```
L1-001: [Title] ──depends on──> nothing (start here)
L1-002: [Title] ──depends on──> L1-001
L1-003: [Title] ──can parallel with──> L1-002
```

If there are no Level 1 changes needed, explicitly state: "No system-level architecture changes required. The current deployment, communication, and data architecture are appropriate for the current and projected workload. Proceed to Level 2."

---

## Level 2: Cross-Cutting Concern Changes

These changes affect the system **horizontally** — across all or most components. They are not localized to a single service or module. They must be designed as a cohesive strategy before implementing per-component.

Pull all findings from Part 3 Sections 2–5 and Section 1e that represent **system-wide concerns**:

- **Security hardening:** Authentication/authorization model changes, secrets management overhaul, encryption-at-rest/in-transit rollout, input validation strategy. These must be designed once and applied consistently — not invented independently per component.
- **Scalability infrastructure:** Caching strategy, connection pooling, state externalization, async processing infrastructure. These require shared infrastructure (Redis, message broker, etc.) that must be provisioned before individual components can use it.
- **Resilience framework:** Circuit breaker library selection, retry policy standardization, timeout strategy, dead letter queue setup. These should be implemented as shared middleware or libraries, not duplicated per service.
- **Observability platform:** Structured logging format, correlation ID propagation, distributed tracing setup, monitoring dashboards, alerting rules. These require platform-level decisions (OpenTelemetry? Application Insights? Datadog?) before individual components can instrument.

For each Level 2 task:

| Field | Description |
|---|---|
| **Task ID** | L2-001, L2-002, etc. |
| **Title** | Clear, action-oriented title |
| **Source findings** | Which Part 3 section(s) and finding(s) this addresses |
| **Business impact** | What user-facing, revenue, or operational outcome does this change serve? What happens to the business if we skip this? |
| **Who else should care** | Besides engineering — product, security, compliance, operations, support? Why? |
| **What changes** | Describe the cross-cutting change |
| **Scope** | Which components are affected — list them explicitly |
| **Design decision** | The system-wide standard or approach being established (e.g., "All services will use OpenTelemetry for tracing with correlation IDs propagated via HTTP headers") |
| **Shared infrastructure required** | What needs to be provisioned or configured centrally (e.g., Redis cluster, message broker, monitoring platform) |
| **Per-component rollout** | How this will be applied to individual components — big bang or incremental? Which components first and why? |
| **Depends on** | Task IDs (including Level 1 tasks if applicable) |
| **Blocks** | Task IDs |
| **Can parallel with** | Tasks at the same level that can run simultaneously |
| **Validation criteria** | How you know this is done system-wide |
| **Rollback plan** | How to undo if it fails |
| **Effort estimate** | Quick win (hours) / Moderate (days) / Significant (weeks) / Major (sprints) |

### Level 2 Execution Sequence

After listing all Level 2 tasks, produce a sequenced execution order with the same format as Level 1.

**Important:** Some Level 2 tasks may depend on Level 1 tasks (e.g., you cannot add circuit breakers to a communication path that is being replaced in Level 1). Make these cross-level dependencies explicit.

If there are no Level 2 changes needed, explicitly state: "No cross-cutting changes required. Security, scalability, resilience, and observability are already implemented consistently across the system. Proceed to Level 3."

---

## Level 3: Per-Component Refactoring

This is where the bulk of the hands-on work happens. **Every component that has findings from Part 3 gets a single, consolidated work package.**

### Step 1: Component Finding Matrix

First, produce a matrix that maps every Part 3 finding to the component it affects:

| Component | Section 1 (Architecture) | Section 2 (Security) | Section 3 (Scalability) | Section 4 (Resilience) | Section 5 (Observability) | Section 6 (Code Structure) | Section 7 (Code Quality) |
|---|---|---|---|---|---|---|---|
| [Component A] | Finding 1a.1 | Finding 2.3 | — | Finding 4.1 | — | Finding 6.2 | Finding 7.1, 7.4 |
| [Component B] | — | Finding 2.1 | Finding 3.2 | — | Finding 5.1 | — | Finding 7.2 |

This matrix exists so that **no finding is lost** in the merge. Every Part 3 finding must appear somewhere in this matrix.

### Step 2: Consolidated Work Packages

For each component with findings, produce a single work package:

| Field | Description |
|---|---|
| **Task ID** | L3-001, L3-002, etc. |
| **Component** | Name of the component or module |
| **All findings addressed** | List every Part 3 finding being tackled in this work package (e.g., "1a.1, 2.3, 4.1, 6.2, 7.1, 7.4") |
| **Business impact** | What does this component do for the business? What user-facing or operational outcome improves when this work package is complete? What happens to the business if we skip it? |
| **Who else should care** | Besides engineering — product, security, compliance, operations, support? Why? |
| **Internal execution order** | The order to address findings WITHIN this component, and why. (Architecture changes first → security → resilience → code structure → code quality) |
| **What changes** | Describe the full scope of changes to this component |
| **Current state** | How the component works today |
| **Target state** | How it should work after all changes |
| **Migration approach** | How to get from current to target — incremental or rewrite? Can it be done behind a feature flag? |
| **Tests required** | What tests need to be written or updated to validate the changes (unit, integration, e2e) |
| **Depends on** | Task IDs (Level 1 and Level 2 tasks that must complete first, plus any Level 3 sibling tasks) |
| **Blocks** | Task IDs |
| **Can parallel with** | Other Level 3 work packages that can run simultaneously |
| **Validation criteria** | How you know this component is done |
| **Rollback plan** | How to undo if it fails |
| **Effort estimate** | Quick win (hours) / Moderate (days) / Significant (weeks) / Major (sprints) |

### Level 3 Execution Sequence

After listing all Level 3 work packages, produce a sequenced execution order. Group work packages that can be parallelized:

```
Wave 1 (parallel): L3-001 [Component A], L3-003 [Component C]
Wave 2 (parallel): L3-002 [Component B], L3-004 [Component D] ──depends on──> L3-001
Wave 3: L3-005 [Component E] ──depends on──> L3-002, L3-004
```

If there are no component-level changes needed, explicitly state: "No per-component refactoring required. All architectural and cross-cutting changes fully address the Part 3 findings. Proceed to Level 4."

---

## Level 4: Housekeeping

These changes do not affect system behavior — they improve maintainability, reduce risk, and pay down non-functional debt. They are safe to execute at any time after Levels 1–3, and many can be parallelized freely.

Pull all remaining findings from Part 3 Sections 7–9 that were NOT already addressed in Level 3 work packages:

- **Dependency updates:** Outdated packages, known CVEs, deprecated libraries, dependency weight reduction.
- **Dead code removal:** Unused files, functions, imports, variables, feature flags for features that shipped long ago.
- **Documentation:** Missing setup guides, stale docs flagged by the trust rules, missing ADRs for decisions made during Levels 1–3, API documentation gaps.
- **Code hygiene:** Inconsistent naming, TODO/FIXME cleanup, linter configuration, formatting standardization.

For each Level 4 task:

| Field | Description |
|---|---|
| **Task ID** | L4-001, L4-002, etc. |
| **Title** | Clear, action-oriented title |
| **Source findings** | Which Part 3 finding(s) this addresses |
| **Business impact** | Why does this housekeeping matter? Connect to developer velocity, onboarding speed, security posture, or operational risk — not just "cleaner code." What happens if we skip it? |
| **What changes** | Describe the change |
| **Risk level** | Low (cosmetic) / Medium (dependency update could break) / High (removing code that might still be needed) |
| **Depends on** | Usually "None" or specific Level 3 tasks if the component was refactored |
| **Can parallel with** | Most Level 4 tasks can run in parallel — state explicitly |
| **Validation criteria** | How you know this is done |
| **Effort estimate** | Quick win (hours) / Moderate (days) / Significant (weeks) |

### Level 4 Execution Sequence

Group Level 4 tasks by type (dependencies, dead code, documentation, hygiene) and indicate which can be parallelized.

If there are no housekeeping changes needed, explicitly state: "No housekeeping required. All findings have been addressed in Levels 1–3."

---

## Execution Summary

After all 4 levels are defined, produce a high-level summary:

### Total Task Count

| Level | Tasks | Estimated Effort |
|---|---|---|
| Level 1: System Architecture | [count] | [total effort range] |
| Level 2: Cross-Cutting Concerns | [count] | [total effort range] |
| Level 3: Per-Component Refactoring | [count] work packages | [total effort range] |
| Level 4: Housekeeping | [count] | [total effort range] |
| **Total** | **[count]** | **[total effort range]** |

### Critical Path

Identify the longest dependency chain from start to finish — the critical path that determines the minimum time to complete all work:

```
L1-001 → L2-003 → L3-002 → L3-005
[estimated total: X weeks]
```

### Quick Wins

List tasks across all levels that can be completed in hours with no dependencies — the team can start these immediately to build momentum:

| Task ID | Title | Level | Effort | Business Impact |
|---|---|---|---|---|
| L4-001 | [Example] | 4 | Hours | [Why this quick win matters to the business] |
| L2-001 | [Example] | 2 | Hours | [Why this quick win matters to the business] |

### Risk Register

Summarize the highest-risk tasks across all levels:

| Task ID | Title | Risk | Business Consequence | Mitigation |
|---|---|---|---|---|
| L1-001 | [Example] | [What could go wrong] | [What happens to users/revenue/operations if this risk materializes] | [How to mitigate] |

### Business Alignment Summary

Produce a summary that maps the execution plan back to business outcomes. A stakeholder reading only this section should understand why this work matters:

1. **Top 3 business outcomes this plan delivers** — What measurable improvements will the business see when this plan is complete? (e.g., "Checkout reliability improves from ~97% to ~99.9% during peak hours," "New feature deployment time drops from 2 weeks to 2 days," "Security posture meets SOC 2 compliance requirements")
2. **What the business risks by not doing this work** — The cost of inaction in business terms, not technical terms. Lost revenue, user churn, compliance violations, inability to scale for a planned launch, mounting support costs.
3. **What the business can do after this work that it cannot do today** — New capabilities unlocked. Features that were previously impossible, markets that can now be entered, SLAs that can now be offered.

---

## Findings Traceability Check

Before finalizing, verify completeness:

1. **Open Part 3's Section 10 (Prioritized Action Plan).** Every item listed there must appear in this execution plan.
2. **Open the Component Finding Matrix (Level 3 Step 1).** Every Part 3 finding must appear in at least one cell.
3. **If any finding is missing**, add it to the appropriate level and explain why it was initially omitted.

State explicitly: "All [N] findings from Part 3 have been accounted for in this execution plan." Or: "The following findings were intentionally excluded: [list] because [reason]."

---

**Apply all rules from `docs/360°-repo-takeover-trust-rules.md` before generating output.**

---

## OUTPUT FORMAT

Save the output as `ownership-pre-audit-findings/REFACTORING_EXECUTION_PLAN.md`. The `ownership-pre-audit-findings/` folder should already exist from previous steps. If it does not, create it.

Use the exact structure defined above for each level. Keep task IDs consistent (L1-001, L2-001, L3-001, L4-001) so they can be cross-referenced throughout the document.

Write in plain language. For every task, ensure a developer who was NOT part of the assessment can pick up the task, understand what to do, and execute it without needing to re-read the full Part 3 audit. For every task, ensure a non-technical stakeholder can read the business impact field and understand why this work matters without understanding the technical details.
