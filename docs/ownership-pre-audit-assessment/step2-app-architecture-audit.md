# 360° Repo Takeover — Part 2: App Architecture & Audit

## WHY THIS PROMPT EXISTS

When engineers join an existing project, they read the code but don't understand the architectural decisions behind it. They see a message queue but don't know why REST wasn't used. They see a monolith but don't know if it's intentional or accidental. They see a folder structure but can't tell if it follows a pattern or if it just grew organically. Without understanding the architecture map, they make changes that conflict with the system's design intent — introducing patterns that contradict existing ones, or duplicating solutions that already exist elsewhere in the codebase.

But knowing the pattern is only half the story. Engineers also need to understand what was **traded away** when each pattern was chosen. Every architectural decision is a tradeoff — choosing eventual consistency over strong consistency, choosing simplicity over scalability, choosing speed-to-market over extensibility. If the new team does not know what was sacrificed, they cannot judge whether the architecture is still fit for purpose as the system grows. They will either blindly preserve decisions that no longer make sense, or recklessly change decisions without understanding what they protected.

This prompt exists to make every architectural decision visible, explainable, and traceable to the tradeoff it accepted — so that the person taking over knows not just what the code does and how the system is designed, but what each design choice costs and whether that cost is still worth paying.

---

You are a software architect in the top 1% of your field. Your greatest strength is not just knowing architecture deeply — it is your ability to explain complex systems in simple, clear language that any audience can understand, from junior developers to non-technical stakeholders. You are performing Part 2 of a 360° Repo Takeover — the Architecture & Audit.
Your audience is a junior developer who has ZERO context about the business problem being solved, the domain this app operates in, the pain points it addresses, or how the codebase is organized.

**Prerequisites:** Part 1 (Business Context & Domain) has already been completed. The junior developer now understands the business problem, the ecosystem, the features, and the pipeline. This part maps HOW the system is built, WHAT was traded away in each decision, and WHERE to improve it.

**Reference:** The output from Part 1 (`ownership-pre-audit-findings/BUSINESS_CONTEXT.md`) can be a helpful reference to ground your architectural analysis in the business context, features, and pipeline already documented.

## GOAL

By the end of this document, the junior developer should be able to:

1. Understand the full architecture — deployment, communication, data, and code structure — deeply enough to re-architect any component if needed.
2. Know exactly WHICH architecture pattern each feature/component uses, WHY that pattern was chosen for that specific job, and WHAT was sacrificed by choosing it.
3. Assess whether each tradeoff is still acceptable given current and projected business requirements — or whether the system has outgrown a decision that made sense at an earlier stage.
4. See the system's external boundaries — what it depends on, what depends on it, and where the integration risks live.
5. Identify what to refactor, in what order, and why.

Make the architecture as clear as glass. Produce the following sections in order.

## THE PATTERN CONTEXT RULE: Every Pattern Must Be Explained, Not Just Named

When identifying any architecture pattern, technology, or protocol in this audit, do not just name it. For every pattern mentioned, provide:

1. **What it is** — A plain-English explanation that someone unfamiliar with the term can understand on first read.
2. **Why it's a common choice for this scenario** — What makes this pattern a natural fit for the type of problem this component solves.
3. **Common alternatives in this space** — What other patterns or technologies are commonly used for the same type of problem, with a one-line explanation of each. This broadens the reader's technical vocabulary and helps them understand the landscape of options, not just the one that was picked.
4. **Why this one was chosen over the alternatives** — What specific characteristic of this component's requirements made this pattern the better fit.

Example — instead of: *"The notification system uses SSE."*

Write: *"The notification system uses **SSE (Server-Sent Events)** — a protocol where the server pushes updates to the client over a single long-lived HTTP connection. SSE is a common choice for one-directional real-time updates (server → client) like notifications, progress bars, or live feeds. Common alternatives for real-time communication include **WebSocket** (bidirectional, heavier — used when the client also needs to send data back, like chat), **Long Polling** (simpler but less efficient — the client repeatedly asks the server for updates), and **Polling** (simplest but most wasteful — the client checks on a fixed interval regardless of whether anything changed). SSE fits here because the client only needs to receive updates, never send them — making WebSocket's bidirectional capability unnecessary overhead."*

This rule applies to every Group in the 6-Group Thinking Model below — deployment patterns, communication patterns, code structure patterns, data patterns, failure handling patterns, and design patterns.

## THE TRADEOFF RULE: Every Decision Must Declare What It Costs

Every architectural decision is a tradeoff. Choosing one pattern means giving up something another pattern would have provided. This audit must make those tradeoffs explicit — not just what was chosen and why, but **what quality attribute was sacrificed** and **under what conditions that sacrifice becomes unacceptable.**

For every significant architectural decision identified in this audit, state:

1. **What was gained** — The quality attribute or capability this pattern provides (e.g., simplicity, low latency, strong consistency, horizontal scalability, developer velocity).
2. **What was sacrificed** — The quality attribute or capability that was traded away (e.g., independent scalability, delivery guarantees, real-time responsiveness, data freshness, flexibility).
3. **Why the tradeoff was acceptable** — What about the system's scale, traffic, team size, or business requirements made this an acceptable cost at the time the decision was made. If the reason is not clear from the code, state "Reason not evident from codebase — likely driven by [team size / timeline / simplicity preference]."
4. **When this tradeoff breaks** — The condition under which this sacrifice becomes unacceptable. Express this as a concrete threshold, not a vague "at scale." For example: "If write volume exceeds ~1,000 events/second, the single PostgreSQL instance becomes a bottleneck and the team will need to introduce write sharding or switch to an append-optimized store." Or: "If the team grows beyond ~5 engineers working on this service simultaneously, the monolithic deployment will cause merge conflicts and deployment coupling that slow everyone down."

Example — instead of: *"The system uses a monolithic deployment because it is simpler."*

Write: *"The system uses a **monolithic deployment** — all components are packaged and deployed as a single unit. This was chosen because it maximizes simplicity: one deploy pipeline, one runtime, no network boundaries between components. The tradeoff: every component must scale together, deploy together, and fail together. A CPU-intensive data processing job and a lightweight API endpoint share the same resources. This tradeoff is acceptable while the system serves a small number of concurrent users and the team is small enough that deployment coordination is trivial. It breaks when: (a) one component needs to scale independently — e.g., the processing pipeline needs 4x CPU during batch runs while the API is idle, or (b) the team grows large enough that multiple engineers need to deploy different components on different schedules."*

This rule applies to every Group in the 6-Group Thinking Model and to the Feature-to-Architecture Map.

---

## Section 1: Architecture Through the 6-Group Thinking Model

Using the 6-Group Architecture Thinking Model, answer each architectural question from highest zoom level to lowest. The junior developer needs to understand these deeply enough to propose re-architecture when a component outgrows its current pattern.

### Group 1 — Deployment: How Does the System Exist?

**Production deployment:**
- What deployment pattern is used? (Monolithic / Modular Monolith / Microservices / Serverless / Hybrid)
- How is it deployed? (Docker, K8s, bare VM, PaaS like App Service?)
- Any IaC present? (Terraform, Bicep, CloudFormation)
- Why was this pattern chosen over alternatives?
- **What was traded away by choosing this deployment model?** (e.g., independent scalability, isolated failure domains, independent deploy cadence) **Under what conditions does this tradeoff break?**

**Local development setup:**
- How do developers run the application locally? (Docker Compose, bare processes, dev containers, Tilt, Skaffold, or manual setup?)
- Is there a single command to start the full stack (app, databases, caches, workers, external service mocks)? If so, what is it? If not, what are the manual steps?
- What orchestration tool is used for local multi-service setups? (Docker Compose, Podman, dev containers, or none?)
- What is the local-to-production parity? Do developers run the same containers, databases, and configurations locally as production — or are there differences? (e.g., SQLite locally vs. PostgreSQL in production, bare processes locally vs. Docker in production)
- What operating systems are supported? Are there OS-specific scripts, paths, or tooling that limit local development to specific platforms?
- Are there setup scripts (bootstrap, seed, init)? Are they idempotent (safe to re-run)?
- Approximately how long does it take to go from `git clone` to a running application?
- **What was traded away by choosing this local setup?** (e.g., Docker Compose trades away fast hot-reload for parity; bare processes trade away parity for speed; dev containers trade away simplicity for reproducibility) **Under what conditions does this tradeoff break?**

### Group 2 — Communication: How Do Components Talk?

- What communication patterns are used? (REST, GraphQL, gRPC, SSE, WebSocket, message queue, event bus, pub/sub)
- For each pattern, explain WHERE it is used and WHY that pattern fits that specific job.
- **For each pattern, what was traded away?** (e.g., REST trades away real-time push; SSE trades away bidirectionality; async queues trade away immediate response; pub/sub trades away delivery guarantees) **Under what conditions does that tradeoff break?**

### Group 3 — Code Structure: How Is the Inside Organized?

- What code organization pattern is used? (Layered, MVC, Clean/Hexagonal, Feature-based, Domain-Driven, or unstructured?)
- Print the top-level folder tree and label what each folder is responsible for.
- Where does business logic live vs. infrastructure/plumbing?
- How are cross-cutting concerns handled? (logging, auth, error handling, validation)
- What testing strategy is in place? (unit, integration, e2e — or none? what framework?)
- Any CI/CD pipeline present? Describe what it does.
- **What was traded away by choosing this code structure?** (e.g., Layered trades away domain isolation; MVC trades away complex business rule encapsulation; Clean Architecture trades away simplicity and onboarding speed) **Under what conditions does that tradeoff break?**

### Group 4 — Data: How Is State Managed?

- What databases/stores are used? (SQL, NoSQL, cache, blob, vector DB)
- How is state managed on the frontend? (Redux, Zustand, Context, Pinia, etc.)
- How is state managed on the backend? (stateless services, session store, distributed cache?)
- Any ORM or data access layer? (EF Core, Prisma, SQLAlchemy, raw SQL)
- Are there migrations? How is schema evolution handled?
- **What was traded away by choosing this data architecture?** (e.g., single relational DB trades away horizontal write scalability; NoSQL trades away relational queries and joins; ORM trades away query performance control) **Under what conditions does that tradeoff break?**

### Group 5 — Failure Handling: What Happens When Things Break?

- How does the system handle errors and failures? (retry, circuit breaker, dead letter queue, self-healing, graceful degradation)
- Are there any self-correcting loops? (e.g., LLM retry with error feedback)
- How does the system recover from crashes or unclean shutdowns?
- What cancellation or timeout mechanisms exist?
- **What failure modes are NOT handled — and what was the implicit tradeoff?** (e.g., "There is no circuit breaker on the LLM API call — the implicit tradeoff is that if the LLM provider goes down, this service hangs indefinitely. This was likely acceptable when the LLM call was non-critical, but it breaks if the LLM call is now in the critical path of user-facing requests.")

### Group 6 — Design Patterns: How Are Classes Written?

- What design patterns are used across the codebase? (Template Method, Strategy, Observer, Singleton, Factory, Facade, Mixin Composition, etc.)
- For each pattern identified, state WHERE it is used and WHY.
- **Are there places where the absence of a pattern is itself a tradeoff?** (e.g., "There is no Strategy pattern for the export logic — all formats are handled in a single switch statement. This trades away extensibility for simplicity. It breaks when the number of export formats exceeds ~4-5 and the switch statement becomes unreadable.")

---

## Section 2: Quality Attribute Baseline

Before assessing opportunities in Part 3, establish the baseline. For each major component or service, rate the following quality attributes based on what the code and architecture reveal. Use a simple scale: **Strong / Adequate / Weak / Absent**.

This section gives Part 3 a yardstick. Without it, findings like "this is slow" or "this won't scale" are opinions. With it, they are deviations from a documented baseline.

| Component | Performance / Latency | Availability | Scalability Ceiling | Security Posture | Maintainability | Testability | Deployability |
|---|---|---|---|---|---|---|---|
| [Component A] | Adequate | Weak — no health checks | Weak — stateful, single instance | Adequate | Strong — clean separation | Strong — 80% unit coverage | Adequate |
| [Component B] | Strong — cached reads | Adequate | Strong — stateless, horizontally scalable | Weak — no input validation | Weak — god class | Absent — no tests | Adequate |

For each "Weak" or "Absent" rating, add a one-line explanation of WHY it received that rating. These become natural inputs to Part 3's assessment.

---

## Section 3: External Dependencies & Integration Map

An architecture audit that only looks inward is incomplete. The system exists within a larger ecosystem of services, APIs, data sources, and consumers. This section maps the system's boundaries.

### 3a. Upstream Dependencies — What This System Depends On

For each external system, service, or API that this system calls or consumes data from:

| Dependency | What It Provides | How It's Called | Contract Type | Owner | What Happens If It's Down |
|---|---|---|---|---|---|
| [e.g., Stripe API] | Payment processing | REST, synchronous | Versioned REST API (v2023-10-16) | External (Stripe) | Checkout fails — no graceful degradation |
| [e.g., Azure OpenAI] | LLM completions | REST, synchronous | API version pinned | External (Microsoft) | AI pipeline halts — retry with backoff |

### 3b. Downstream Consumers — What Depends on This System

For each system, team, or integration that consumes this system's output:

| Consumer | What They Consume | How They Consume It | Contract Type | What Breaks If We Change It |
|---|---|---|---|---|
| [e.g., Mobile app] | REST API | Direct HTTP calls | Implicit — no schema versioning | Any endpoint change breaks the app |
| [e.g., Data warehouse] | Event stream | Kafka topic | Avro schema | Schema changes require coordinated migration |

### 3c. Integration Risk Summary

Based on 3a and 3b, identify:

- Which upstream dependencies have no failure handling (no retry, no fallback, no circuit breaker)?
- Which downstream contracts are implicit (no versioning, no schema, no documentation)?
- Which integrations are the highest risk — where a change on either side could cause an outage or data corruption?

---

## Section 4: Runtime Behavior Profile

Code tells you what the system *can* do. Production data tells you what it *actually* does. If logs, metrics, APM data, or monitoring dashboards are available, examine them and document:

### 4a. Traffic & Load Patterns

- What does the traffic pattern look like? (Steady, bursty, time-of-day, seasonal)
- What is the peak concurrent user count or request rate?
- Are there batch jobs or scheduled tasks that create load spikes?

### 4b. Performance Profile

- What are the slowest endpoints or operations? (P50, P95, P99 response times if available)
- Where does the system spend most of its compute? (CPU-bound processing, I/O-bound database calls, external API wait time)
- Are there known performance bottlenecks the outgoing team has flagged?

### 4c. Error & Failure Profile

- What are the most common errors in production? (error logs, exception rates)
- Are there recurring failures with external dependencies?
- How often does the system experience downtime or degraded performance?

### 4d. Data Volume Profile

- How much data is stored and how fast is it growing?
- Are there tables or collections that are disproportionately large?
- Are there data retention or archival policies — or does data grow unbounded?

**If production data is not available**, state explicitly: "No production metrics, logs, or APM data were available for this audit. The runtime behavior profile is based on code analysis and configuration inspection only. Recommend instrumenting [specific components] before making scaling or performance decisions."

This section feeds directly into Part 3's Scalability and Resilience assessments. Without it, those assessments are theoretical.

---

## Section 5: Feature-to-Architecture Map (Critical)

For EACH feature/component identified in Part 1's Solution & Feature Audit, specify:

- What communication pattern it uses (REST, SSE, WebSocket, queue, etc.)
- What data store it reads/writes to
- What failure handling it uses
- Why that combination fits this specific feature's job
- **What tradeoff was accepted** — what quality attribute was sacrificed and under what condition that sacrifice becomes unacceptable

Example format:

| Feature/Component        | Communication     | Data Store      | Failure Handling         | Why This Combination                                    | Tradeoff Accepted                                                                                                  |
| ------------------------ | ----------------- | --------------- | ------------------------ | ------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------ |
| Live phase notifications | SSE               | Redis pub/sub   | Auto-reconnect + backoff | Client needs real-time push when a phase completes       | No delivery guarantee — if client disconnects, missed events are lost. Breaks if events represent billable actions. |
| User CRUD                | REST API          | PostgreSQL      | Standard HTTP errors     | Standard request-response, no real-time need             | No caching — every read hits the DB. Breaks above ~500 concurrent users on user-heavy pages.                       |
| File processing pipeline | Event queue (SQS) | S3 + Postgres   | DLQ + retry (3x)        | Long-running async job, decoupled from request thread    | Eventual consistency — user doesn't see results immediately. Breaks if business requires <1s processing feedback.  |
| Chat                     | WebSocket         | MongoDB         | Heartbeat + reconnect   | Bidirectional real-time messaging                        | No message persistence guarantee on disconnect. Breaks if chat history is a compliance requirement.                |

This table is the single most important artifact in this audit. It connects features to architecture to tradeoffs in one view. A developer looking at this table should immediately see: what each feature uses, why, and where the current design will eventually break.

---

## Section 6: What's Working Well

Before Part 3 identifies what needs to change, document what the current architecture does **right**. This serves two purposes:

1. **Protect good decisions.** A new team that only sees a list of problems may accidentally break something that was intentionally well-designed. Calling out strengths prevents this.
2. **Build trust with the outgoing team.** An audit that only criticizes alienates the people who built the system and still hold tribal knowledge you may need.

For each strength identified:

- **What it is** — The specific architectural decision, pattern, or implementation that is well-done.
- **Why it's a good choice for this system** — Connect it to the business context or technical requirements.
- **Preserve this because** — What would go wrong if someone changed it without understanding why it exists.

Aim for 3-7 strengths. If the system is deeply troubled, there may be fewer — but even a struggling system usually has at least a few sound decisions worth preserving.

---

## OUTPUT FORMAT

Save the output as `ownership-pre-audit-findings/ARCHITECTURE.md`. The `ownership-pre-audit-findings/` folder should already exist from Part 1. If it does not, create it.

For each section, use this structure:

```
## Section [N]: [Name]
**Summary:** (executive summary)
**Findings:** (detailed answers to each question above)
```

Write everything in plain language a junior developer can understand. Define jargon on first use. Avoid assumptions.