# 360° Repo Takeover — Part 3: Assessment & Opportunities

## WHY THIS PROMPT EXISTS

After understanding the business context and architecture, the natural next question is "what should we improve?" But most teams skip the strategic assessment and jump straight to fixing whatever annoys them first — a messy file here, an outdated dependency there. They refactor code inside a component whose architecture is fundamentally wrong, wasting effort on walls when the foundation needs repair. They fix naming conventions while a security hole sits unnoticed. Nobody talks about this, but it is one of the most common ways engineering time is wasted after a project handover. This prompt exists to force a top-down assessment: evaluate architecture before code, security before features, scalability before cosmetics — and give every finding a clear "why it matters" so engineers prioritize with confidence instead of guessing.

---

You are a software architect in the top 1% of your field. Your greatest strength is not just knowing architecture deeply — it is your ability to explain complex systems in simple, clear language that any technical audience can understand. 

Your audience is **engineers and architects** — the people who will own, maintain, and improve this codebase. They are technical, but they may be new to this specific system. Write for someone who understands software engineering concepts but has no prior context about this repo's design decisions, trade-offs, or technical debt.

**Prerequisites:** Part 1 (Business Context & Domain) and Part 2 (Architecture & Audit) have already been completed. You now know WHY this app exists and HOW it is built. This part identifies WHAT needs to change and in WHAT ORDER — from the highest-impact architectural decisions down to code-level improvements.

**Helpful references from previous steps:**
- `ownership-pre-audit-findings/BUSINESS_CONTEXT.md` — Can help connect findings back to the business impact and clarify what user growth or data sensitivity looks like.
- `ownership-pre-audit-findings/ARCHITECTURE.md` — This documents the **current** architecture as it exists today — not an ideal or target state. Use it as the baseline for your assessment. Every opportunity you identify in this audit should be measured against what is actually in place, not what the architecture should theoretically be.

## THE CORE PRINCIPLE: Fix the Foundation Before the Walls

If a building's foundation is cracked, you do not repaint the walls. The same applies to software.

If a component's **architecture** is wrong — synchronous when it should be async, monolithic when it should be decoupled, missing a caching layer, using the wrong data store — then fixing code-level issues inside that component is wasted effort. You will rewrite that code the moment you fix the architecture.

This audit works **top-down**: system-level architecture first, then security, then scalability, then resilience, then observability, then code structure, then code quality. Each level is assessed before the next because findings at a higher level can invalidate work at a lower level.

## THE EXPLANATION RULE: Every Finding Must Answer "Why Does This Matter?"

Do not just state what is wrong. Even a senior engineer receiving a finding like "this service is synchronous" gains nothing unless they understand **why it matters in this specific system, for this specific business problem.**

For EVERY finding in this audit, follow this structure:

1. **What is the issue** — State the problem clearly.
2. **Why it matters for this app specifically** — Connect it to the business context, user experience, or operational risk from Part 1. Not generic textbook reasons — specific to THIS system.
3. **What happens if you ignore it** — Paint the consequence. "If user load doubles, this component becomes a bottleneck because..." or "If this service fails, there is no recovery path, which means..."
4. **What the fix looks like** — Describe the direction, not a full implementation. Enough for the developer to understand the approach.
5. **Severity** — Critical / High / Medium / Low.

The goal: a developer reading any finding should immediately understand not just WHAT to fix, but WHY it is worth their time — and be able to explain that reasoning to a non-technical stakeholder.

---

## GOAL

By the end of this document, the developer should be able to:

1. Identify every architecture-level opportunity before touching any code.
2. Understand WHY each issue matters — not just that it exists.
3. See the dependency chain between findings (fix X before Y).
4. Have a prioritized, strategic action plan ordered by impact.

---

## Section 1: Architecture Opportunities

Using the 6-Group Architecture Thinking Model (see `architecture/architecture-patterns-thinking-model.md`), assess each architectural dimension from Part 2's findings. For each group, evaluate whether the current pattern is the right fit — or whether the system has outgrown it.

### 1a. Deployment Opportunities

Part 2 identified the deployment pattern (Monolith / Modular Monolith / Microservices / Serverless / Hybrid). Now assess:

- Is the deployment pattern still the right fit for the current and projected workload? Has the system outgrown it?
- Are there components that need to scale independently but are locked into a shared deployment? (e.g., a CPU-intensive pipeline deployed alongside a lightweight API)
- Are there components deployed as separate services that do not need to be? (e.g., a microservice that always deploys with another — a distributed monolith in disguise)
- Is the deployment infrastructure (Docker, K8s, PaaS, serverless) appropriate for the workload profile — or is it over/under-engineered?
- Are there components that would benefit from a different deployment model? (e.g., bursty event-triggered work that should be serverless instead of always-on)

For each finding, follow the Explanation Rule above.

### 1b. Communication Opportunities

Part 2 identified which communication patterns are used (REST, SSE, WebSocket, message queue, pub/sub, etc.) and where. Now assess:

- For each component or interaction, is the communication pattern the right fit for that specific job?
- Are there synchronous paths that should be asynchronous? (e.g., a user request that triggers a long-running process but makes the user wait for the full result instead of returning immediately and processing in the background)
- Are there asynchronous paths that should be synchronous? (e.g., an event-driven flow for something that needs an immediate response)
- Are there real-time features built on polling instead of WebSocket/SSE?
- Are there fan-out scenarios (one event, many receivers) using point-to-point calls instead of pub/sub?
- Are there exactly-once processing requirements using pub/sub instead of a message queue?
- Is there a mismatch between what the user experience requires and how the communication is implemented?

Map each component to its current communication pattern and state whether it is correct or needs to change:

| Component / Interaction | Current Pattern | Right Fit? | Recommended Change (if any) | Why |
|---|---|---|---|---|

For each finding, follow the Explanation Rule above.

### 1c. Code Structure Opportunities

Part 2 identified how each service or module is organized internally (Layered, MVC, Clean/Hexagonal, DDD, etc.). Now assess:

- Is the code organization pattern appropriate for the complexity of each component? (e.g., Clean Architecture for a simple CRUD service is over-engineering; Layered for a service with complex business rules is under-engineering)
- Are there components where business logic leaks into infrastructure layers? (e.g., business rules inside API controllers, database queries inside service logic)
- Are there components where the structure has degraded over time — the pattern is nominally there but boundaries have been violated?
- Is there a component that started simple but grew complex enough to warrant a structural upgrade? (e.g., Layered → Hexagonal because external dependencies are now swappable)

For each finding, follow the Explanation Rule above.

### 1d. Data Opportunities

Part 2 identified the data patterns in use (CRUD, Repository, CQRS, Event Sourcing). Now assess:

- Are read and write patterns balanced, or has the system evolved into heavy-read / heavy-write scenarios that CRUD cannot efficiently serve?
- Are there components that would benefit from CQRS — separate read and write models — because reads vastly outnumber writes or need a different shape?
- Are there missing data access abstractions? (e.g., direct SQL scattered across services instead of a Repository pattern, making database swaps impossible)
- Are there missing caching layers for frequently-read, rarely-written data?
- Is the data store itself the right fit? (e.g., relational DB for document-shaped data, or NoSQL for highly relational data)
- Are there N+1 query patterns, unbounded queries, or missing indexes that will become bottlenecks at scale?

For each finding, follow the Explanation Rule above.

### 1e. Failure Handling Opportunities

Part 2 identified how the system handles failures (retry, circuit breaker, bulkhead, saga, DLQ). Now assess:

- For every external call (APIs, databases, third-party services, LLMs), is there failure handling in place — or does a failure cascade uncontrolled?
- Are retries bounded with backoff, or do they risk hammering a struggling dependency?
- Are there multi-step processes that can leave data in an inconsistent state if interrupted? Do they need a Saga pattern?
- Are there missing circuit breakers on calls to unreliable or slow external services?
- Are failed messages or jobs silently lost, or is there a dead letter queue for inspection and retry?
- If the system uses LLMs or AI pipelines: are there self-correcting loops? What stops them from looping forever?

For each finding, follow the Explanation Rule above.

### 1f. Design Pattern Opportunities

Part 2 identified the code-level design patterns in use (Factory, Strategy, Observer, Builder, Adapter, etc.). Now assess:

- Are there places where design patterns are missing and would reduce complexity? (e.g., nested if/else chains that should be a Strategy pattern, manual object construction that should be a Builder)
- Are there places where patterns are over-applied? (e.g., Factory for a class that only has one implementation, Observer for a single subscriber)
- Are there tightly coupled dependencies that should use an Adapter or Port interface for swappability?
- Are there god classes or utility files that should be decomposed using appropriate patterns?

For each finding, follow the Explanation Rule above.

---

## Section 2: Security Assessment

Assess the security posture of the system. This is not a penetration test — it is an architectural security review.

- **Authentication & Authorization:** How are users authenticated? Is authorization enforced at every layer or only at the API gateway? Are there endpoints that bypass auth? Is there role-based or attribute-based access control, and is it consistently applied?
- **Secrets Management:** Are secrets (API keys, connection strings, tokens) stored securely? Or are they hardcoded, committed to git, or stored in plaintext config files?
- **Data Protection:** Is sensitive data encrypted at rest and in transit? Are there PII fields stored without encryption or masking? Are database connections using TLS?
- **Input Validation:** Is user input validated and sanitized at system boundaries? Are there SQL injection, XSS, or command injection vectors?
- **Dependency Vulnerabilities:** Are there known CVEs in the dependency tree? Are dependencies pinned or floating?
- **Attack Surface:** What is exposed to the internet? Are there unnecessary open ports, debug endpoints, or admin panels without auth?

**Best Practices & Opportunities:** For each finding, do not just state the problem — explain the industry best practice or proven approach that addresses it. For example: if secrets are hardcoded, explain what a secrets management solution looks like (Azure Key Vault, AWS Secrets Manager, environment variable injection via CI/CD) and why that approach is the standard. Give the engineer a clear picture of what "good" looks like so they know the target, not just the gap.

For each finding, follow the Explanation Rule above.

---

## Section 3: Scalability Assessment

Assess whether the system can handle growth — in users, data volume, and feature complexity.

- **Stateful vs. Stateless:** Are services stateless and horizontally scalable? Or is state stored in-memory, preventing scaling beyond a single instance?
- **Database Scalability:** Can the data layer handle 10x current load? Are there missing indexes, N+1 query patterns, or unbounded queries?
- **Concurrency:** How does the system handle concurrent requests? Are there race conditions, lock contention, or shared mutable state?
- **Async vs. Sync:** Are long-running operations blocking request threads? Should any synchronous paths be converted to async (queues, background workers, event-driven)?
- **Caching:** Are there hot paths that hit the database on every request when they could be cached? Is there a caching strategy, or is it ad-hoc?
- **Resource Limits:** Are there unbounded loops, unlimited file uploads, or queries without pagination that could exhaust memory or CPU?

**Best Practices & Opportunities:** For each finding, explain the proven scalability pattern or technique that solves it. For example: if services are stateful, explain how to externalize state to Redis or a distributed cache and what that migration looks like. If the database cannot handle 10x load, explain whether the opportunity is read replicas, connection pooling, query optimization, caching layer, or CQRS — and why that specific approach fits this system's access patterns. The engineer should walk away knowing not just "this won't scale" but "here is the path to make it scale."

For each finding, follow the Explanation Rule above.

---

## Section 4: Resilience & Failure Handling

Assess what happens when things break — because they will.

- **Error Propagation:** Do errors cascade? If one service fails, does it take down upstream services?
- **Retry & Recovery:** Are there retry mechanisms for transient failures (network timeouts, rate limits, temporary unavailability)? Are retries bounded to prevent retry storms?
- **Circuit Breakers:** Are there circuit breakers on calls to external services or databases?
- **Graceful Degradation:** Can the system continue operating (even partially) when a dependency is down? Or does one failure cause a total outage?
- **Data Integrity:** Are there operations that can leave data in an inconsistent state if interrupted mid-way? Are there transactions or compensation mechanisms?
- **Timeout & Cancellation:** Do long-running operations have timeouts? Can users cancel in-progress work?
- **Self-Correcting Loops:** If the system uses LLMs or AI pipelines, are there retry-with-feedback loops? What stops them from looping forever?

**Best Practices & Opportunities:** For each finding, explain the resilience pattern that addresses it and how it would be implemented in this system. For example: if errors cascade, explain the bulkhead or circuit breaker pattern and where specifically it should be added. If multi-step operations can leave inconsistent state, explain whether a Saga, outbox pattern, or transactional boundary is the right fit and why. The engineer should understand the specific resilience technique, not just that "failure handling is missing."

For each finding, follow the Explanation Rule above.

---

## Section 5: Observability

Assess whether the team can understand what the system is doing in production.

- **Logging:** Is there structured logging? Are logs useful for debugging, or are they generic "error occurred" messages? Are there correlation IDs to trace a request across services?
- **Monitoring:** Are there health checks, uptime monitors, or dashboards? Would the team know if the system went down before users reported it?
- **Alerting:** Are there alerts for critical failures, high error rates, or resource exhaustion?
- **Tracing:** For multi-service systems, is there distributed tracing (OpenTelemetry, Jaeger, etc.)?
- **Audit Trail:** For systems handling sensitive data or financial transactions, is there an audit log of who did what and when?

**Best Practices & Opportunities:** For each finding, explain what production-grade observability looks like for this type of system. For example: if there are no correlation IDs, explain how to implement them (middleware that generates a request ID and propagates it through headers and log context) and what tools support it (OpenTelemetry, Application Insights, etc.). If monitoring is absent, explain what a minimal viable monitoring setup looks like — health endpoints, uptime checks, error rate dashboards — and what platform fits the existing infrastructure.

For each finding, follow the Explanation Rule above.

---

## Section 6: Code Structure & Design

Now — and only now — zoom into code-level organization. Architecture-level issues have been addressed above. This section focuses on how the code within each component is structured.

- **Separation of Concerns:** Is business logic cleanly separated from infrastructure (HTTP handlers, database queries, external service calls)? Or is everything tangled together?
- **Coupling & Cohesion:** Are modules tightly coupled (changing one requires changing many)? Are responsibilities scattered across unrelated files?
- **Design Patterns:** Are patterns used appropriately? Are there missing patterns that would simplify the code? (e.g., a Strategy pattern where there are nested if/else chains, a Repository pattern where SQL is scattered across services)
- **Abstraction Level:** Are there god classes/files doing too many things? Are there premature abstractions that add complexity without value?
- **Naming & Conventions:** Are names consistent and descriptive? Can a new developer understand what a function does from its name?
- **Testability:** Is the code structured to be testable? Are dependencies injectable, or are they hardcoded?

**Best Practices & Opportunities:** For each finding, explain what the well-structured version would look like. For example: if business logic is tangled with database queries, show what extracting it into a service layer or use case layer looks like and why it makes the code testable. If a god class exists, explain how to decompose it — which responsibilities to extract, what pattern to use (Facade, Strategy, separate services), and in what order. The engineer should see a clear before/after picture for each structural issue.

For each finding, follow the Explanation Rule above.

---

## Section 7: Code Quality & Hygiene

The lowest level — individual code-level issues.

- **Dead Code:** Files, functions, imports, or variables that are never used.
- **Duplication:** Logic that is copy-pasted across multiple places instead of being extracted.
- **Complexity:** Functions that are too long, too nested, or doing too many things.
- **Hardcoded Values:** Magic numbers, hardcoded URLs, or inline config that should be externalized.
- **TODO/FIXME/HACK Comments:** Unfinished work that was left behind.
- **Inconsistencies:** Mixed coding styles, inconsistent error handling patterns, different approaches to the same problem in different files.

**Best Practices & Opportunities:** For each finding, explain the cleanup technique and what the clean version looks like. For example: if there is duplication, explain where to extract the shared logic (utility function, base class, shared module) and show the approach. If functions are too complex, explain how to decompose them — extract method, guard clauses, early returns — and what makes a function the right size. For inconsistencies, recommend a linter configuration or coding standard that would prevent them going forward.

For each finding, follow the Explanation Rule above.

---

## Section 8: Dependency Health

Assess the health of the project's dependencies.

- **Outdated Packages:** Which dependencies are significantly behind their latest versions?
- **Known Vulnerabilities:** Are there CVEs in the dependency tree?
- **Deprecated APIs:** Is the code using APIs or libraries that are deprecated or end-of-life?
- **Dependency Weight:** Are there heavy dependencies being used for trivial functionality?
- **Lock File Hygiene:** Is the lock file committed? Are dependency versions pinned or floating?

**Best Practices & Opportunities:** For each finding, explain the recommended migration path or alternative. For example: if a library is deprecated, name the recommended replacement and what the migration effort looks like. If there are CVEs, explain which ones are exploitable in this context vs. which are theoretical. For dependency weight, suggest lighter alternatives or explain when to replace a library with a small utility function. Recommend a dependency management strategy (Dependabot, Renovate, `pip-audit`, `npm audit`) appropriate for this project's ecosystem.

For each finding, follow the Explanation Rule above.

---

## Section 9: Documentation Gaps

Assess the state of documentation — but remember: documentation that is wrong is worse than no documentation.

- **Missing Documentation:** What critical information has no documentation at all? (setup instructions, architecture decisions, API contracts, environment variables, deployment process)
- **Stale Documentation:** What existing documentation contradicts the current codebase? (Apply the Documentation Trust Rule from `docs/360°-repo-takeover-trust-rules.md`)
- **Onboarding Gaps:** Could a new developer set up the project and make their first contribution using only the existing documentation? If not, what is missing?

**Best Practices & Opportunities:** For each finding, explain what the documentation should contain and the recommended format. For example: if setup instructions are missing, outline what a good setup guide covers (prerequisites, environment variables, install steps, first run, common errors). If API docs are absent, recommend whether OpenAPI/Swagger auto-generation, Postman collections, or manually maintained docs are the right fit for this project's API style. If architecture decisions are undocumented, recommend ADRs (Architecture Decision Records) and what the first few ADRs should cover based on the findings in this audit.

For each finding, follow the Explanation Rule above.

---

## Section 10: Prioritized Action Plan

Now synthesize everything above into a single, ordered action plan. Group findings by priority:

### Critical — Fix immediately (blocks everything else or is a live risk)
List findings that represent security vulnerabilities, data loss risks, or architectural blockers that prevent other improvements.

### High — Fix before building new features
List findings that will cause increasing pain if ignored — scalability limits, missing resilience, architectural mismatches.

### Medium — Fix during normal development
List findings that improve maintainability and developer experience — code structure, observability, documentation.

### Low — Fix opportunistically
List findings that are nice-to-have — code hygiene, naming, minor cleanup.

For each item in the plan:
- Reference which section it came from (e.g., "Section 2: Secrets stored in plaintext")
- State the dependency chain if applicable (e.g., "Fix this BEFORE addressing the scaling issue in Section 3")
- Estimate effort: Quick win (hours) / Moderate (days) / Significant (weeks)

---

**Apply all rules from `docs/360°-repo-takeover-trust-rules.md` before generating output.**

---

## OUTPUT FORMAT

Save the output as `ownership-pre-audit-findings/ASSESSMENT_AND_OPPORTUNITIES.md`. The `ownership-pre-audit-findings/` folder should already exist from Part 1. If it does not, create it.

For each section, use this structure:

```
## Section [N]: [Name]
**Summary:** (executive summary — 2-3 sentences on the overall state of this area)
**Findings:**

### Finding [N.1]: [Title]
- **What:** [The issue]
- **Why it matters:** [Specific to this app and business context]
- **If ignored:** [The consequence]
- **Fix direction:** [The approach]
- **Severity:** [Critical / High / Medium / Low]
```

Write everything in plain language that anyone — developers, business analysts, stakeholders — can understand. For every technical finding, explain why it matters in terms a non-technical person can grasp. Define jargon on first use. Avoid assumptions.
