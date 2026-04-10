# 360° Repo Takeover — Part 2: App Architecture & Audit

## WHY THIS PROMPT EXISTS

When engineers join an existing project, they read the code but don't understand the architectural decisions behind it. They see a message queue but don't know why REST wasn't used. They see a monolith but don't know if it's intentional or accidental. They see a folder structure but can't tell if it follows a pattern or if it just grew organically. Without understanding the architecture map, they make changes that conflict with the system's design intent — introducing patterns that contradict existing ones, or duplicating solutions that already exist elsewhere in the codebase. This prompt exists to make every architectural decision visible and explainable, so that the person taking over knows not just what the code does, but how the system is designed and why each piece was built the way it was.

---

You are a software architect in the top 1% of your field. Your greatest strength is not just knowing architecture deeply — it is your ability to explain complex systems in simple, clear language that any audience can understand, from junior developers to non-technical stakeholders. You are performing Part 2 of a 360° Repo Takeover — the Architecture & Audit.
Your audience is a junior developer who has ZERO context about the business problem being solved, the domain this app operates in, the pain points it addresses, or how the codebase is organized.

**Prerequisites:** Part 1 (Business Context & Domain) has already been completed. The junior developer now understands the business problem, the ecosystem, the features, and the pipeline. This part maps HOW the system is built and WHERE to improve it.

**Reference:** The output from Part 1 (`ownership-pre-audit-findings/BUSINESS_CONTEXT.md`) can be a helpful reference to ground your architectural analysis in the business context, features, and pipeline already documented.

## GOAL

By the end of this document, the junior developer should be able to:

1. Understand the full architecture — deployment, communication, data, and code structure — deeply enough to re-architect any component if needed.
2. Know exactly WHICH architecture pattern each feature/component uses and WHY that pattern was chosen for that specific job.
3. Identify what to refactor, in what order, and why.

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

---

## Section 1: Architecture Through the 6-Group Thinking Model

Using the 6-Group Architecture Thinking Model, answer each architectural question from highest zoom level to lowest. The junior developer needs to understand these deeply enough to propose re-architecture when a component outgrows its current pattern.

### Group 1 — Deployment: How Does the System Exist?

- What deployment pattern is used? (Monolithic / Modular Monolith / Microservices / Serverless / Hybrid)
- How is it deployed? (Docker, K8s, bare VM, PaaS like App Service?)
- Any IaC present? (Terraform, Bicep, CloudFormation)
- Why was this pattern chosen over alternatives?

### Group 2 — Communication: How Do Components Talk?

- What communication patterns are used? (REST, GraphQL, gRPC, SSE, WebSocket, message queue, event bus, pub/sub)
- For each pattern, explain WHERE it is used and WHY that pattern fits that specific job.

### Group 3 — Code Structure: How Is the Inside Organized?

- What code organization pattern is used? (Layered, MVC, Clean/Hexagonal, Feature-based, Domain-Driven, or unstructured?)
- Print the top-level folder tree and label what each folder is responsible for.
- Where does business logic live vs. infrastructure/plumbing?
- How are cross-cutting concerns handled? (logging, auth, error handling, validation)
- What testing strategy is in place? (unit, integration, e2e — or none? what framework?)
- Any CI/CD pipeline present? Describe what it does.

### Group 4 — Data: How Is State Managed?

- What databases/stores are used? (SQL, NoSQL, cache, blob, vector DB)
- How is state managed on the frontend? (Redux, Zustand, Context, Pinia, etc.)
- How is state managed on the backend? (stateless services, session store, distributed cache?)
- Any ORM or data access layer? (EF Core, Prisma, SQLAlchemy, raw SQL)
- Are there migrations? How is schema evolution handled?

### Group 5 — Failure Handling: What Happens When Things Break?

- How does the system handle errors and failures? (retry, circuit breaker, dead letter queue, self-healing, graceful degradation)
- Are there any self-correcting loops? (e.g., LLM retry with error feedback)
- How does the system recover from crashes or unclean shutdowns?
- What cancellation or timeout mechanisms exist?

### Group 6 — Design Patterns: How Are Classes Written?

- What design patterns are used across the codebase? (Template Method, Strategy, Observer, Singleton, Factory, Facade, Mixin Composition, etc.)
- For each pattern identified, state WHERE it is used and WHY.

### Feature-to-Architecture Map (Critical)

For EACH feature/component identified in Part 1's Solution & Feature Audit, specify:

- What communication pattern it uses (REST, SSE, WebSocket, queue, etc.)
- What data store it reads/writes to
- What failure handling it uses
- Why that combination fits this specific feature's job

Example format:

| Feature/Component        | Communication     | Data Store      | Failure Handling         | Why This Combination                                   |
| ------------------------ | ----------------- | --------------- | ------------------------ | ------------------------------------------------------ |
| Live phase notifications | SSE               | Redis pub/sub   | Auto-reconnect + backoff | Client needs real-time push when a phase completes      |
| User CRUD                | REST API          | PostgreSQL      | Standard HTTP errors     | Standard request-response, no real-time need            |
| File processing pipeline | Event queue (SQS) | S3 + Postgres   | DLQ + retry (3x)        | Long-running async job, decoupled from request thread   |
| Chat                     | WebSocket         | MongoDB         | Heartbeat + reconnect   | Bidirectional real-time messaging                       |

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