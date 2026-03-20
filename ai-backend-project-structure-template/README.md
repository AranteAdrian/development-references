# Enterprise AI Backend Template

A production-grade project structure for Python + FastAPI + AI/LLM backend applications. Built from patterns validated across legitimate sources including FastAPI best-practices repositories, Cookiecutter Data Science (7.6k+ stars, 8+ years in production use), production LangGraph templates, and enterprise architecture guides from Towards Data Science, The New Stack, and real company engineering blogs.

---

## Who is this for?

This template is purpose-built for **AI-powered backend systems**: RAG applications, multi-agent platforms, ML-powered APIs, conversational AI services, document intelligence backends, and any project where you are orchestrating LLMs with tools, data, and guardrails behind a FastAPI surface.

**This is NOT a general-purpose backend template.** If you are building a pure CRUD API, a data engineering ETL pipeline, a microservice proxy, or a non-Python backend, half of these folders will sit empty — and empty structure is misleading structure. Use a simpler template for those.

---

## Scenario: Building "InsightBot"

To make every folder concrete, we will walk through the entire template using a single realistic project: **InsightBot** — an enterprise AI assistant that lets employees ask questions about internal company documents (HR policies, product specs, financial reports). It uses RAG to retrieve relevant documents, an agent to reason over them, and guardrails to prevent leaking sensitive data.

We will follow the exact order a developer would encounter these folders when building InsightBot from scratch — starting from project setup, through core infrastructure, into the AI-specific layers, and finally into testing and deployment.

---

## Full Structure

```
enterprise-ai-backend-template/
│
├── app/                          # Application package (all source code)
│   ├── main.py                   # FastAPI app factory and startup
│   ├── core/                     # App-wide config, security, constants
│   │   ├── config.py
│   │   ├── security.py
│   │   ├── exceptions.py
│   │   └── constants.py
│   ├── db/                       # Database engine, session, base model
│   │   ├── session.py
│   │   └── base.py
│   ├── models/                   # SQLAlchemy ORM models (database tables)
│   ├── schemas/                  # Pydantic models (API request/response shapes)
│   ├── repositories/             # Data access layer (between services and DB)
│   ├── api/                      # HTTP layer
│   │   └── v1/
│   │       ├── router.py         # Aggregates all v1 endpoint routers
│   │       ├── dependencies.py   # Shared FastAPI dependencies (auth, pagination)
│   │       └── endpoints/        # Per-resource route files (chat.py, documents.py)
│   ├── services/                 # Business logic layer
│   ├── agents/                   # LLM agent definitions and graphs
│   ├── pipelines/                # Multi-step AI workflows (RAG, ingestion)
│   ├── tools/                    # Functions agents can invoke
│   ├── prompts/                  # Versioned prompt templates
│   ├── guardrails/               # Input/output validation and safety
│   ├── memory/                   # Agent memory and conversation state
│   ├── retrievers/               # RAG retrieval logic (vector search, reranking)
│   ├── middlewares/               # Request logging, CORS, tracing, rate limiting
│   └── workers/                  # Background tasks (Celery, async jobs)
│
├── alembic/                      # Database migration scripts
│   ├── env.py
│   └── versions/
├── tests/                        # All test code
│   ├── conftest.py               # Shared fixtures across all tests
│   ├── unit/                     # Fast, isolated, no external deps
│   ├── integration/              # Slower, tests real endpoint + DB behavior
│   └── e2e/                      # Full-flow tests across the entire stack
├── evals/                        # AI evaluation datasets and harness
│   ├── datasets/                 # Golden Q&A pairs and test inputs
│   ├── runners/                  # Scripts that execute evaluations
│   └── results/                  # Eval output scores and reports
├── data/                         # Local data artifacts
│   ├── raw/                      # Original, unmodified source data
│   ├── processed/                # Cleaned, chunked, ready-to-embed data
│   └── vectors/                  # Locally embedded vectors for RAG development
├── notebooks/                    # Jupyter notebooks for exploration only
├── scripts/                      # CLI utilities, one-off migration scripts
├── .github/                      # CI/CD — GitHub Actions (if using GitHub)
│   └── workflows/
├── .azdo/                        # CI/CD — Azure DevOps (if using Azure DevOps)
│   ├── templates/                # Reusable pipeline templates per deploy target
│   │   ├── backend-container-app.yml
│   │   ├── frontend-container-app.yml
│   │   └── function-app.yml
│   └── variables/                # Environment-specific variable groups
│       ├── dev.yml
│       ├── staging.yml
│       └── production.yml
│
├── azure-pipelines.yml           # Main Azure DevOps pipeline entry point (at root)
├── alembic.ini                   # Alembic migration config
├── pyproject.toml                # Project metadata and dependencies
├── Dockerfile                    # Container build instructions
├── .dockerignore                 # Files excluded from Docker build context
├── docker-compose.yml            # Local multi-service orchestration
├── Makefile                      # Developer command shortcuts
├── .pre-commit-config.yaml       # Automated pre-commit quality checks
├── .env.example                  # Template for environment variables
├── .gitignore                    # Files excluded from version control
└── CONTRIBUTING.md               # Guide for adapting and contributing to this template
```

---

## Step-by-Step Walkthrough

### Phase 1: Project Foundation

These are the files and folders you set up before writing a single line of business logic. They establish how the project is configured, how dependencies are managed, and how the team runs things locally.

---

#### `pyproject.toml`

**What it is:** The single source of truth for your project's metadata, dependencies, and tool configuration (linter rules, test settings, build config).

**InsightBot example:** This is where you declare that InsightBot depends on `fastapi`, `sqlalchemy`, `langchain-core`, `langgraph`, etc. Your dev dependencies (pytest, ruff) go here too under `[project.optional-dependencies]`.

**Why it matters:** Replaces the old `requirements.txt` + `setup.py` + `setup.cfg` trio. Every modern Python project (pip, uv, poetry) reads from this file. If your project doesn't have one, dependency management will fragment across multiple files and confuse every new team member.

📎 **Reference:** [Python Packaging — Writing pyproject.toml](https://packaging.python.org/en/latest/guides/writing-pyproject-toml/)

---

#### `.env.example`

**What it is:** A template showing every environment variable the application needs, with placeholder values. The actual `.env` file (which contains real secrets) is never committed to git.

**InsightBot example:** Contains placeholders for `DATABASE_URL`, `OPENAI_API_KEY`, `SECRET_KEY`, `REDIS_URL`, etc. A new developer clones the repo, copies `.env.example` to `.env`, fills in their local values, and they're running.

**Why it matters:** Without this, onboarding a new developer means Slack messages asking "what env vars do I need?" Every production FastAPI template includes this — it's the cheapest documentation you can write.

📎 **Reference:** [Pydantic Settings — Environment Variables](https://docs.pydantic.dev/latest/concepts/pydantic_settings/)

---

#### `.gitignore`

**What it is:** Tells git which files and folders to exclude from version control.

**InsightBot example:** Excludes `.env` (secrets), `__pycache__/` (bytecode), `.venv/` (virtual env), `data/raw/*` (large files), `*.pyc`, `.DS_Store`, and IDE folders.

**Why it matters:** Accidentally committing a `.env` file with API keys is a real security incident. Accidentally committing `data/raw/` with 2GB of PDFs breaks the repo for everyone. This file is your first line of defense.

📎 **Reference:** [Git — gitignore Documentation](https://git-scm.com/docs/gitignore)

---

#### `Makefile`

**What it is:** A collection of shortcut commands that standardize how the team runs, tests, lints, and deploys the project.

**InsightBot example:** Instead of remembering `uvicorn app.main:app --reload --host 0.0.0.0 --port 8000`, the developer types `make run`. Instead of `pytest tests/unit -x --timeout=10`, they type `make test-unit`. Common targets: `make run`, `make test`, `make test-unit`, `make test-integration`, `make lint`, `make format`, `make migrate`, `make docker-up`.

**Why it matters:** The presence of a Makefile is a recognized signal of engineering discipline in production projects. It eliminates "works on my machine" problems by standardizing every command. Its absence in a project is a red flag for experienced engineers.

📎 **Reference:** [Cookiecutter Data Science — Makefile Conventions](https://cookiecutter-data-science.drivendata.org/)

---

#### `Dockerfile`

**What it is:** Instructions for building a container image of your application.

**InsightBot example:** Starts from `python:3.11-slim`, installs dependencies from `pyproject.toml`, copies the app code, and sets the entrypoint to `uvicorn app.main:app`. The production image runs with multiple Uvicorn workers behind a process manager.

**Why it matters:** Containers are the standard deployment unit for enterprise backends. Without a Dockerfile, your deployment process is manual and environment-dependent. With one, InsightBot runs identically on a developer's laptop, in staging, and in production.

📎 **Reference:** [Docker — Dockerfile Reference](https://docs.docker.com/reference/dockerfile/)

---

#### `docker-compose.yml`

**What it is:** Defines all the services InsightBot needs to run locally — the app, database, Redis, and optionally a Celery worker — in a single file.

**InsightBot example:** Defines `app` (FastAPI), `db` (PostgreSQL), and `redis` (for caching and task queue). A developer runs `make docker-up` and the entire local environment starts with correct networking between services.

**Why it matters:** AI backends almost always depend on multiple services (database, vector store, cache, message broker). Docker Compose lets every developer spin up an identical local environment in one command. No more "install Postgres on your machine" documentation.

📎 **Reference:** [Docker Compose Documentation](https://docs.docker.com/compose/)

---

#### `.dockerignore`

**What it is:** Tells Docker which files and folders to exclude when building the image. Works the same way as `.gitignore` but specifically controls what gets copied into the Docker build context.

**InsightBot example:** Without this file, building the InsightBot image copies everything — the `.git/` folder, all raw PDFs in `data/raw/`, notebooks, the virtual environment, and local `.env` files — into the image. With `.dockerignore`, only the application code and dependencies are included, keeping the image lean and ensuring no local secrets or data files accidentally end up in a container that gets pushed to a registry.

**Why it matters:** A missing `.dockerignore` is one of the most common Docker mistakes in AI projects specifically — because AI projects tend to have large data folders that have no business being in a production image. It also prevents cache invalidation: if Docker sees the `.git/` folder changed (which it always does on every commit), it invalidates the entire build cache even when no application code changed.

📎 **Reference:** [Docker — .dockerignore File](https://docs.docker.com/build/concepts/context/#dockerignore-files)

---

#### `.pre-commit-config.yaml`

**What it is:** Configuration for pre-commit hooks — automated checks that run on your local machine every time you run `git commit`, before the commit is saved.

**InsightBot example:** When an InsightBot developer runs `git commit`, pre-commit automatically runs `ruff check` for linting and `ruff format` for formatting. If either fails, the commit is blocked and the developer sees the errors immediately in their terminal — no push, no CI run, no waiting. After fixing, they commit again and it goes through.

**Why it matters:** CI catches quality issues after the code is already in the repo. Pre-commit catches them before the commit even exists — a faster and cleaner feedback loop. It is the local counterpart to your CI pipeline, not a replacement for it. FastAPI's own repository and Tiangolo's full-stack template both ship with this file.

📎 **Reference:** [pre-commit — Documentation](https://pre-commit.com/)

---

### Phase 2: Application Core (`app/core/`, `app/db/`)

These folders contain the infrastructure that every other part of the application depends on. They are the foundation — built first, changed rarely.

---

#### `app/main.py`

**What it is:** The entry point of the FastAPI application. Creates the app instance, registers middleware, includes routers, and defines the health check endpoint.

**InsightBot example:** Creates the FastAPI app, adds CORS middleware (so the frontend can call the API), adds the request logging middleware, and mounts the v1 API router. The server is started by pointing Uvicorn at `app.main:app`.

**Why it matters:** This file should be thin — 30 to 50 lines. If `main.py` has business logic, route definitions, or database queries in it, the project has no separation of concerns. Its only job is to wire things together.

📎 **Reference:** [FastAPI — Bigger Applications with Multiple Files](https://fastapi.tiangolo.com/tutorial/bigger-applications/)

---

#### `app/core/config.py`

**What it is:** Application settings loaded from environment variables using Pydantic's `BaseSettings`. Defines every configurable value: database URL, API keys, model names, rate limits, feature flags.

**InsightBot example:** `PROJECT_NAME`, `DATABASE_URL`, `OPENAI_API_KEY`, `DEFAULT_MODEL` (e.g. "gpt-4o"), `MAX_TOKENS`, `ALLOWED_ORIGINS`, `LOG_LEVEL`. All read from `.env` with sensible defaults for local development.

**Why it matters:** Hardcoded config (API keys in source code, model names buried in a function) is the most common security and maintenance failure in AI projects. Pydantic settings gives you type validation, `.env` file support, and a single place to see every knob your application exposes.

📎 **Reference:** [Pydantic Settings Management](https://docs.pydantic.dev/latest/concepts/pydantic_settings/)

---

#### `app/core/security.py`

**What it is:** Authentication and authorization utilities — password hashing, JWT token creation and verification, OAuth helpers.

**InsightBot example:** Functions like `hash_password()`, `verify_password()`, `create_access_token()`. These are used by the auth service and the API dependency layer, never called directly from routes.

**Why it matters:** Security logic scattered across multiple files is how vulnerabilities happen. Centralizing it means one place to audit, one place to update when you rotate algorithms.

📎 **Reference:** [FastAPI — Security Tutorial](https://fastapi.tiangolo.com/tutorial/security/)

---

#### `app/core/exceptions.py`

**What it is:** Custom exception classes for the application. Defines domain-specific errors that the API layer catches and translates into proper HTTP status codes.

**InsightBot example:** `EntityNotFoundError` (→ 404), `PermissionDeniedError` (→ 403), `LLMProviderError` (→ 502), `GuardrailViolationError` (→ 422). The service layer raises these; the API layer handles them.

**Why it matters:** Without custom exceptions, your services end up raising generic `ValueError` or `Exception`, and your routes are full of `try/except` blocks guessing what went wrong. Custom exceptions create a clean contract between layers.

📎 **Reference:** [FastAPI — Handling Errors](https://fastapi.tiangolo.com/tutorial/handling-errors/)

---

#### `app/core/constants.py`

**What it is:** Application-wide constants — values that are fixed and do not change between environments.

**InsightBot example:** `MAX_AGENT_ITERATIONS = 10`, `DEFAULT_CHUNK_SIZE = 1000`, `DEFAULT_CHUNK_OVERLAP = 200`, `MAX_TOKENS_PER_REQUEST = 8192`. These are not config (they don't change per environment) — they are engineering decisions baked into the application.

**Why it matters:** Constants scattered as magic numbers across your codebase are a maintenance nightmare. "Why is this 1000?" is a question nobody should have to ask. Centralizing them with clear names makes intent explicit.

📎 **Reference:** [fastapi-best-practices — Project Structure Conventions](https://github.com/zhanymkanov/fastapi-best-practices)

---

#### `app/db/session.py`

**What it is:** Creates the database engine and async session factory. Provides the `get_db` dependency that FastAPI routes use to get a database session.

**InsightBot example:** Sets up an async SQLAlchemy engine pointing at PostgreSQL, creates a session factory, and defines a `get_db()` async generator that handles commit/rollback lifecycle automatically.

**Why it matters:** Database session management is infrastructure — it belongs in its own module, not inside `models/` (circular imports) or `core/` (wrong responsibility). Every production FastAPI guide separates this.

📎 **Reference:** [SQLAlchemy — Async Session](https://docs.sqlalchemy.org/en/20/orm/extensions/asyncio.html)

---

#### `app/db/base.py`

**What it is:** The SQLAlchemy declarative base class that all ORM models inherit from.

**InsightBot example:** A single `Base` class. Every model in `app/models/` inherits from it. Alembic reads this base's metadata to auto-detect schema changes.

**Why it matters:** Keeping the base separate from models prevents circular imports. The base is imported by both the models (to inherit from) and by Alembic's `env.py` (to read metadata). If it lived inside a model file, importing it would pull in all models and their dependencies.

📎 **Reference:** [SQLAlchemy — Declarative Mapping](https://docs.sqlalchemy.org/en/20/orm/mapping_styles.html#declarative-mapping)

---

### Phase 3: Data Layer (`app/models/`, `app/schemas/`)

These two folders define the shape of your data — how it's stored in the database and how it flows through your API.

---

#### `app/models/`

**What it is:** SQLAlchemy ORM model classes. Each file represents a database table (or a closely related group of tables).

**InsightBot example:** `conversation.py` defines the `Conversation` and `Message` tables. `document.py` defines the `Document` table (storing metadata about ingested company docs). `user.py` defines the `User` table.

**Why it matters:** Models are your database schema as code. They must be separate from Pydantic schemas because they serve different purposes — models define storage, schemas define API contracts. Mixing them couples your API surface to your database structure, making both harder to change independently.

📎 **Reference:** [SQLAlchemy ORM — Mapped Classes](https://docs.sqlalchemy.org/en/20/orm/quickstart.html)

---

#### `app/schemas/`

**What it is:** Pydantic models that define the exact shape of data entering and leaving your API endpoints. Request schemas validate input; response schemas control what gets returned.

**InsightBot example:** `chat.py` contains `ChatRequest` (what the frontend sends: message text, conversation ID) and `ChatResponse` (what comes back: assistant message, source citations, token count). `document.py` contains `DocumentUploadRequest` and `DocumentStatusResponse`. Internal DTOs like `AgentContext` (passed between pipeline stages) also live here.

**Why it matters:** This is the most universally agreed-upon separation in FastAPI production projects. Every legitimate source — FastAPI's own docs, the fastapi-best-practices repo (zhanymkanov), and enterprise architecture guides — separates schemas from models. Without this folder, Pydantic schemas end up scattered across services, routes, and model files, creating import tangles and making your API contract invisible.

📎 **Reference:** [Pydantic — Models Documentation](https://docs.pydantic.dev/latest/concepts/models/)

---

#### `app/repositories/`

**What it is:** The data access layer that sits between services and the database. Each repository class wraps the database queries for one domain entity — abstracting raw SQLAlchemy calls behind a clean interface.

**InsightBot example:** `document_repository.py` contains `get_by_id()`, `list_by_status()`, `save()`, and `delete()` for the `Document` table. `conversation_repository.py` handles fetching and saving conversation history. Services call repositories; repositories are the only layer that knows about SQLAlchemy.

**Why it matters:** Without this layer, services mix business logic with database queries in the same function — making both harder to test and harder to change. With repositories, a service can be tested by passing in a mock repository; the database query logic is tested separately. This is a standard clean architecture pattern and is explicitly demonstrated in production FastAPI templates.

📎 **Reference:** [FastAPI Clean Architecture Example](https://github.com/0xTheProDev/fastapi-clean-example) · [Repository Pattern with FastAPI](https://medium.com/@kacperwlodarczyk/fast-api-repository-pattern-and-service-layer-dad43354f07a)

---

### Phase 4: HTTP Layer (`app/api/`, `app/middlewares/`)

These folders handle everything between the outside world and your business logic — routing requests, validating auth, logging, and rate limiting.

---

#### `app/api/v1/router.py`

**What it is:** The central aggregator that collects all endpoint routers and mounts them under the `/api/v1` prefix.

**InsightBot example:** Imports routers from individual endpoint files (`chat.py`, `documents.py`, `auth.py`) and includes them with appropriate prefixes and tags. When InsightBot adds a "feedback" feature later, the developer creates `feedback.py` with its router and adds one line here.

**Why it matters:** API versioning is a non-negotiable for production APIs. When you need to make breaking changes, you create `v2/` alongside `v1/` and migrate clients gradually. Without versioned routing from day one, you're forced to break existing clients or maintain ugly compatibility hacks.

📎 **Reference:** [FastAPI — APIRouter](https://fastapi.tiangolo.com/tutorial/bigger-applications/#apirouter)

---

#### `app/api/v1/endpoints/`

**What it is:** A subfolder containing one file per API resource. Each file defines the routes for that resource and its own `APIRouter` instance, which `router.py` then imports and includes.

**InsightBot example:** `endpoints/chat.py` defines `POST /chat` and `GET /chat/{id}/history`. `endpoints/documents.py` defines `POST /documents/upload` and `GET /documents/{id}/status`. `endpoints/auth.py` defines `POST /auth/login` and `POST /auth/refresh`. Each file is independently readable and independently testable.

**Why it matters:** Without this subfolder, all routes live in `router.py` — which becomes a hundreds-of-lines monolith as soon as the project grows beyond two resources. The FastAPI official documentation on Bigger Applications explicitly shows this per-resource file pattern as the recommended approach for real projects. Adding a new feature means creating one new file in `endpoints/` and one line in `router.py` — nothing else changes.

📎 **Reference:** [FastAPI — Bigger Applications](https://fastapi.tiangolo.com/tutorial/bigger-applications/)

---

#### `app/api/v1/dependencies.py`

**What it is:** Shared FastAPI dependencies used across multiple endpoints — authentication, database session injection, pagination, current user extraction.

**InsightBot example:** `get_current_user()` dependency that extracts and validates a JWT token from the request header. `get_db()` re-exported for convenience. `PaginationParams` class for list endpoints.

**Why it matters:** FastAPI's dependency injection system is one of its strongest features. Dependencies defined here are reused across all routes, cached per request, and testable in isolation. Without a dedicated file, every route re-implements auth logic.

📎 **Reference:** [FastAPI — Dependencies](https://fastapi.tiangolo.com/tutorial/dependencies/)

---

#### `app/middlewares/`

**What it is:** Cross-cutting concerns that apply to every request — logging, CORS configuration, request tracing (attaching request IDs), rate limiting, and OpenTelemetry integration.

**InsightBot example:** `logging.py` logs every request with method, path, status code, duration, and a unique request ID. `tracing.py` sets up OpenTelemetry spans so you can trace a request from the API layer through the vector DB query into the LLM call and back. This is critical for debugging "why did this request take 12 seconds?"

**Why it matters:** AI backends have uniquely complex observability needs. An LLM call can take 2-30 seconds depending on model, token count, and provider load. Without request-level tracing middleware, debugging performance issues is guesswork. Production AI architecture guides universally recommend OpenTelemetry integration at this layer.

📎 **Reference:** [FastAPI — Middleware](https://fastapi.tiangolo.com/tutorial/middleware/) · [OpenTelemetry Python](https://opentelemetry.io/docs/languages/python/)

---

### Phase 5: Business Logic (`app/services/`)

The bridge between your HTTP endpoints and your AI pipelines.

---

#### `app/services/`

**What it is:** The business logic layer. Services contain the core workflows that connect API endpoints to agents, pipelines, and the database. Routes call services; services call everything else.

**InsightBot example:** `chat_service.py` handles the full lifecycle: receive a user message → look up conversation history → call the RAG pipeline → apply output guardrails → save the response → return it. `document_service.py` handles document upload, chunking, and triggering background ingestion jobs.

**Why it matters:** This is where the "thin controller, fat service" principle lives. Routes should be 5-10 lines: validate input, call service, return response. All real logic lives in services. This makes your business logic testable without needing to spin up an HTTP server — you test services directly with mocked dependencies.

📎 **Reference:** [fastapi-best-practices — Service Layer Pattern](https://github.com/zhanymkanov/fastapi-best-practices)

---

### Phase 6: AI/LLM Layer (`app/agents/`, `app/pipelines/`, `app/tools/`, `app/prompts/`, `app/guardrails/`, `app/memory/`, `app/retrievers/`)

These seven folders are what make this template AI-specific. They do not exist in a standard backend template.

---

#### `app/agents/`

**What it is:** LLM agent definitions — the autonomous reasoning units that use tools and context to answer questions. Each agent has its own system prompt, tool bindings, and execution graph.

**InsightBot example:** `research_agent.py` defines an agent that receives a user question plus retrieved document chunks, reasons about which chunks are relevant, and produces a cited answer. If InsightBot later adds a "summarization agent" or a "code review agent," each gets its own file here.

**Why it matters:** Agents are the most complex and most frequently iterated part of an AI system. Isolating them means you can swap agent frameworks (LangGraph → CrewAI → custom), change model providers, or restructure reasoning logic without touching routes, services, or pipelines.

📎 **Reference:** [LangGraph Documentation](https://langchain-ai.github.io/langgraph/)

---

#### `app/pipelines/`

**What it is:** Multi-step AI workflows that chain together retrieval, transformation, and agent execution. Pipelines are the backbone of your AI features — they define the end-to-end data flow.

**InsightBot example:** `rag_pipeline.py` defines the sequence: embed the user query → search the vector store → rerank results → inject context into the agent → run the agent → validate the output. `ingestion_pipeline.py` defines: receive uploaded PDF → extract text → chunk → embed → upsert into vector store.

**Why it matters:** Without explicit pipelines, the RAG workflow gets smeared across services, agents, and routes — making it impossible to understand, test, or modify the end-to-end flow. A pipeline file is a readable specification of "what happens when a user asks a question," and production ML project guides emphasize that entrypoint scripts for pipelines are the contract that CI/CD relies on.

📎 **Reference:** [Cookiecutter Data Science — Project Structure](https://cookiecutter-data-science.drivendata.org/)

---

#### `app/tools/`

**What it is:** Pure functions that agents can invoke during reasoning. Each tool has a clear name, description (which the LLM reads), and typed inputs/outputs.

**InsightBot example:** `search_tool.py` — searches the vector store. `calculator_tool.py` — performs numerical calculations on financial data. `sql_tool.py` — runs read-only queries against the reporting database. Each tool is a standalone function the research agent can choose to call.

**Why it matters:** Tools are the interface between your agent's reasoning and your application's capabilities. They must be isolated, well-documented, and independently testable. If a tool is buried inside an agent file, it can't be reused by other agents, and its behavior is harder to validate.

📎 **Reference:** [LangChain — Custom Tools](https://python.langchain.com/docs/how_to/custom_tools/)

---

#### `app/prompts/`

**What it is:** Versioned prompt templates used by agents and pipelines. Each prompt has an explicit version identifier so you can roll back without redeploying code.

**InsightBot example:** `research_prompts.py` contains `RESEARCH_AGENT_SYSTEM_V1` (the initial system prompt), then `V2` when you refine it after eval results show hallucination issues. The agent references `V2` by name, and your eval harness can compare V1 vs V2 performance.

**Why it matters:** LLM behavior is part of your API contract. If you change a prompt and performance degrades, you need to roll back immediately. Production AI teams treat prompt versioning as a first-class concern — not comments in code, not git commit messages, but real named versions your application selects at runtime. Without this, you have no way to track what changed when your agent starts behaving differently.

📎 **Reference:** [LangChain — Prompt Templates](https://python.langchain.com/docs/concepts/prompt_templates/)

---

#### `app/guardrails/`

**What it is:** Input and output validation specifically for AI content. Separate from Pydantic schema validation (which handles data shape), guardrails handle content safety — prompt injection detection, PII filtering, output grounding checks, toxicity screening, and policy compliance.

**InsightBot example:** `validators.py` contains `check_input_length()` (reject absurdly long inputs), `detect_prompt_injection()` (catch attempts to override the system prompt), `check_pii_in_output()` (scan agent responses for SSN, credit card, or email patterns before returning to the user), `verify_source_grounding()` (ensure the agent's claims are actually supported by the retrieved documents).

**Why it matters:** This is not optional for enterprise AI. Without output guardrails, InsightBot might leak an employee's salary from an HR document, hallucinate a policy that doesn't exist, or return a response that violates compliance rules. Guardrails are a first-class concern — they deserve their own folder, not buried inside a service.

📎 **Reference:** [OWASP Top 10 for LLM Applications](https://owasp.org/www-project-top-10-for-large-language-model-applications/)

---

#### `app/memory/`

**What it is:** Manages agent memory and conversation state — the mechanism by which an agent knows what was said earlier in a session, retains user preferences across turns, or summarizes long conversations to fit within a context window.

**InsightBot example:** `conversation_memory.py` loads the last N messages from the database and formats them for injection into the agent's context window. `summary_memory.py` compresses old conversation turns into a rolling summary when the history grows too long. `entity_memory.py` tracks key entities mentioned (document names, departments, topics) so the agent can refer back to them without re-reading the full history.

**Why it matters:** Memory is distinct from agent *logic*. An agent decides *what to do*; memory decides *what it knows about the past*. Mixing them inside `agents/` creates files that are both hard to test and hard to swap. LangChain treats memory as a first-class module with its own class hierarchy — `ConversationBufferMemory`, `ConversationSummaryMemory`, `VectorStoreRetrieverMemory` — precisely because it is a separate concern.

📎 **Reference:** [LangChain — Memory Concepts](https://python.langchain.com/docs/concepts/memory) · [LangChain Memory API Reference](https://python.langchain.com/api_reference/langchain/memory.html)

---

#### `app/retrievers/`

**What it is:** The retrieval layer for RAG — responsible for taking a user query, searching the vector store, optionally reranking results, and returning the most relevant document chunks to be injected into the agent's context.

**InsightBot example:** `vector_retriever.py` embeds the user query and runs a similarity search against the vector database. `hybrid_retriever.py` combines dense vector search with BM25 keyword search and merges the results. `reranker.py` takes the top-20 candidates from the vector search and uses a cross-encoder model to reorder them by actual relevance before the top-5 are passed to the agent.

**Why it matters:** Retrieval logic is not a "tool" an agent calls by choice — it is an infrastructure concern that runs before the agent sees any context. Putting it inside `tools/` conflates two different things. Retrieval has its own dependencies (vector store clients, embedding models, reranking models), its own testing strategy (retrieval recall metrics, not agent output quality), and its own optimization lifecycle. LangChain and AWS both define retrievers as a distinct abstraction layer in RAG architectures.

📎 **Reference:** [LangChain — Retrievers](https://python.langchain.com/docs/concepts/retrievers/) · [AWS — RAG Custom Retrievers](https://docs.aws.amazon.com/prescriptive-guidance/latest/retrieval-augmented-generation-options/rag-custom-retrievers.html)

---

### Phase 7: Background Processing (`app/workers/`)

---

#### `app/workers/`

**What it is:** Background task definitions for work that shouldn't block the API response — document ingestion, scheduled eval runs, report generation, batch embedding, cache warming.

**InsightBot example:** `celery_app.py` configures the Celery application connected to Redis. `tasks.py` defines `ingest_documents()` (fetch PDFs, chunk, embed, store — takes minutes, shouldn't block the upload endpoint) and `run_nightly_evals()` (run the eval suite against latest prompts every night).

**Why it matters:** Agent pipelines that include document retrieval and LLM calls can take 5-30 seconds. Some operations (full document ingestion, batch re-indexing) take minutes or hours. Without a task queue, these either block your API (killing throughput) or get hacked into `asyncio.create_task()` calls that lose work if the process restarts. Celery (or any proper task queue) gives you retries, monitoring, and persistence.

📎 **Reference:** [Celery — Getting Started](https://docs.celeryq.dev/en/stable/getting-started/introduction.html)

---

### Phase 8: Database Migrations (`alembic/`)

---

#### `alembic/` and `alembic.ini`

**What it is:** Alembic manages database schema migrations — the versioned, reversible scripts that evolve your database structure over time.

**InsightBot example:** When you add the `Document` table, you run `make migrate-create msg="add documents table"`. Alembic auto-generates a migration script in `alembic/versions/`. When deployed to staging, `make migrate` applies it. If something breaks, `alembic downgrade -1` reverts it.

**Why it matters:** Directly modifying production database schemas with raw SQL is how data loss incidents happen. Migrations give you version control for your database. They live at the project root (not inside `app/`) because Alembic generates its own config structure and fights you if you nest it elsewhere. This is the universal convention in every FastAPI + SQLAlchemy project.

📎 **Reference:** [Alembic Tutorial](https://alembic.sqlalchemy.org/en/latest/tutorial.html)

---

### Phase 9: Testing (`tests/`)

---

#### `tests/conftest.py`

**What it is:** Shared pytest fixtures used across all tests — test client setup, database session fixtures, sample data factories, mock LLM clients.

**InsightBot example:** A `client` fixture that creates an async HTTP client pointed at the test app. A `sample_chat_request` fixture with a realistic message. A `mock_llm` fixture that returns deterministic responses so tests don't hit real OpenAI.

---

#### `tests/unit/`

**What it is:** Fast tests that run in isolation with no external dependencies — no database, no network calls, no LLM API. Test individual functions and classes.

**InsightBot example:** Test that guardrails correctly reject oversized input. Test that prompt templates contain required placeholders. Test that schema validation catches malformed requests. Run with `make test-unit` — should complete in under 10 seconds.

**Why it matters:** Unit tests are your fast feedback loop. They run on every commit in CI. If they're slow (because they hit a database or LLM), developers stop running them.

📎 **Reference:** [pytest Documentation](https://docs.pytest.org/en/stable/)

---

#### `tests/integration/`

**What it is:** Slower tests that verify real behavior across components — actual API endpoints, database operations, pipeline execution with mocked LLM responses.

**InsightBot example:** Test that `POST /api/v1/chat` returns a valid response with correct schema. Test that document upload triggers an ingestion pipeline. Test that the health endpoint returns 200. Run with `make test-integration` — may take up to 2 minutes with a test database.

**Why it matters:** Unit tests verify parts work in isolation; integration tests verify they work together. AI pipelines are especially prone to integration failures — the retriever returns the right chunks but the agent prompt template doesn't format them correctly, or the guardrail blocks a valid response. You need both test types.

---

#### `tests/e2e/`

**What it is:** End-to-end tests that exercise complete user-facing workflows through the entire stack — from HTTP request to database, through the AI pipeline, and back to the HTTP response. Unlike integration tests (which may mock the LLM), e2e tests validate the full system behaves correctly from a user's perspective.

**InsightBot example:** `test_chat_flow.py` sends a real question via the API, lets it flow through the retriever, the agent, and the guardrail layer, and asserts the response contains a valid answer with source citations. `test_document_ingestion.py` uploads a real PDF and asserts that after ingestion, a related question returns chunks from that document.

**Why it matters:** Unit and integration tests can both pass while a real user workflow is broken — because they test components, not the complete journey. E2E tests are the last line of verification before production. They run less frequently (not on every commit, but on every release candidate or nightly), and they are the only tests that can catch issues caused by how all layers interact together end-to-end.

📎 **Reference:** [Twilio — Unit, Integration, and End-to-End Testing: What's the Difference?](https://www.twilio.com/en-us/blog/unit-integration-end-to-end-testing-difference)

---

### Phase 10: AI Evaluation (`evals/`)

---

#### `evals/`

**What it is:** The evaluation harness for your AI system — datasets to test against, scripts to run them, and a place to store results. This folder is what lets you answer "did this change make the AI better or worse?"

**Why it matters:** This is the folder that separates toy projects from production AI. Without evals, you have no way to know if a prompt change, model upgrade, or retriever tweak made things better or worse. Every production ML guide emphasizes that eval harnesses are non-negotiable — they are to AI projects what unit tests are to traditional software.

📎 **Reference:** [OpenAI Evals Framework](https://github.com/openai/evals) · [LangSmith Evaluation](https://docs.smith.langchain.com/)

---

#### `evals/datasets/`

**What it is:** Golden datasets — hand-curated question-and-answer pairs, edge cases, and adversarial inputs used to benchmark the AI system. These are the ground truth your eval runners compare against.

**InsightBot example:** `hr_questions.jsonl` contains 30 questions about HR policies with expected answers and the source document each answer comes from. `adversarial_inputs.jsonl` contains prompt injection attempts and jailbreak queries that the guardrail layer must reject. `regression_cases.jsonl` contains questions that broke in previous versions — ensuring they stay fixed.

**Why it matters:** Without structured datasets, evals are ad hoc and unrepeatable. OpenAI's evals framework stores all datasets in a `data/` registry directory as JSONL files with consistent schema — input, expected output, and metadata. A structured dataset folder means every team member runs evals against the same ground truth.

---

#### `evals/runners/`

**What it is:** Scripts that load a dataset, feed each item through the AI pipeline, collect outputs, and compute scores. Each runner targets a specific pipeline or evaluation dimension.

**InsightBot example:** `run_rag_eval.py` evaluates retrieval quality — for each question in `datasets/hr_questions.jsonl`, it runs the retriever and checks whether the correct source document appears in the top-5 results. `run_answer_eval.py` feeds the full RAG pipeline and uses an LLM-as-judge to score faithfulness and relevance. `run_guardrail_eval.py` tests every adversarial input and asserts the guardrail correctly rejects each one.

**Why it matters:** Separating runners from datasets means you can run different evaluations over the same data — testing retrieval quality independently from answer quality. Each runner is also independently callable from CI or the Celery scheduler.

---

#### `evals/results/`

**What it is:** The output directory for eval runs — score reports, comparison files between prompt versions, and trend data tracked over time.

**InsightBot example:** `2026-03-20_rag_eval.json` contains per-question scores from the RAG eval run on that date. `prompt_v1_vs_v2_comparison.json` shows side-by-side scores when the research agent prompt was updated. These files are checked into version control so the team can see how AI quality has changed across releases.

**Why it matters:** Without stored results, every eval run disappears after you read the terminal output. Persisted results let you compare current performance against a baseline, detect regressions, and build a quality trend over time — which is the entire point of running evals systematically.

---

### Phase 11: Data and Exploration (`data/`, `notebooks/`)

---

#### `data/raw/`

**What it is:** Original, unmodified source data. Files here are never changed after they're placed — they represent the ground truth.

**InsightBot example:** The original HR policy PDFs, product specification documents, and financial reports that InsightBot needs to answer questions about. These are placed here for local development and testing; in production, they live in cloud storage (S3, Azure Blob).

---

#### `data/processed/`

**What it is:** Cleaned, chunked, and transformed data ready for embedding or model consumption.

**InsightBot example:** The PDFs from `raw/` have been parsed, split into 1000-character chunks with 200-character overlap, and saved as JSONL files with metadata. These processed chunks are what the ingestion pipeline embeds and stores in the vector database.

**Why it matters:** Cookiecutter Data Science (the most widely adopted ML project template, with 8+ years of production use) structures data into staged directories — raw, interim, processed — so anyone opening the project can follow the data journey. Mixing raw and processed data in one folder leads to confusion about what's been transformed and what hasn't.

📎 **Reference:** [Cookiecutter Data Science — Opinions on Data](https://cookiecutter-data-science.drivendata.org/)

---

#### `data/vectors/`

**What it is:** Locally stored vector embeddings for RAG development and testing. When you are iterating on chunking strategies, embedding models, or retrieval logic, this is where the embedded output lands before it goes to a hosted vector database.

**InsightBot example:** During local development, `ingest_pipeline.py` embeds the processed document chunks and writes them to `data/vectors/hr_policies.npy` (or a local Qdrant/Chroma instance pointing here). The retriever reads from this directory instead of hitting Pinecone — keeping local development fast and free.

**Why it matters:** Without a designated local vector store location, developers either hit the production vector database during development (expensive, risky) or scatter local embedding files across the project with no consistent convention. This folder makes the RAG development loop self-contained and reproducible.

---

#### `notebooks/`

**What it is:** Jupyter notebooks for exploration, prototyping, and ad-hoc analysis. Notebooks are for exploration only — production code never imports from notebooks.

**InsightBot example:** `01-data-exploration.ipynb` (analyze document lengths and chunk distributions), `02-retriever-comparison.ipynb` (compare BM25 vs dense retrieval quality), `03-prompt-iteration.ipynb` (test prompt variations interactively before promoting to `app/prompts/`).

**Why it matters:** Notebooks are where AI experiments happen. But they have no place in the production code path — they can't be tested, linted, or versioned cleanly. Keeping them in a dedicated folder makes this boundary explicit. The naming convention (number prefix for ordering) comes from Cookiecutter Data Science and is widely adopted.

📎 **Reference:** [Cookiecutter Data Science — Notebooks Convention](https://cookiecutter-data-science.drivendata.org/)

---

### Phase 12: Utilities and DevOps (`scripts/`, `.github/`, `.azdo/`)

---

#### `scripts/`

**What it is:** One-off CLI utilities, data migration scripts, seed scripts, and operational tools that don't belong in the main application.

**InsightBot example:** `seed_demo_data.py` (populate the database with sample conversations for demos), `export_eval_results.py` (export eval scores to a spreadsheet), `backfill_embeddings.py` (re-embed all documents after switching embedding models).

**Why it matters:** These scripts are important but don't belong inside `app/` because they're not part of the running application. They also don't belong in `notebooks/` because they're meant to be run from the command line, not interactively.

---

#### `CONTRIBUTING.md`

**What it is:** A guide for anyone who clones this template — explaining how to adapt it for a real project, what to keep, what to remove, and how to propose improvements back to the template itself.

**InsightBot example:** A new developer clones this template to start InsightBot. The README explains every folder's purpose. `CONTRIBUTING.md` answers the next set of questions: which folders are optional and safe to delete (e.g. `.azdo/` if using GitHub), what to rename first, how to run the full local setup, and how to submit a fix or improvement back to the template if they find a gap.

**Why it matters:** A template without contribution guidance forces every adopter to make the same guesses independently. Since this template is designed to be shared and reused across projects and teams, `CONTRIBUTING.md` is the document that makes that handoff smooth — turning a folder structure into something a team can confidently pick up and run with from day one.

---

### CI/CD: Choose Your Platform

This template includes folder structures for **both** GitHub Actions and Azure DevOps Pipelines. Use whichever your organization runs — delete the other. They solve the same problem (automated lint, test, build, deploy on every push/PR) but have different conventions.

---

#### Option A: `.github/workflows/` (GitHub Actions)

**What it is:** GitHub Actions CI/CD pipeline definitions. GitHub **enforces** this exact path — workflow files must live in `.github/workflows/` or GitHub won't detect them. This is a hard convention.

**InsightBot example:** `ci.yml` defines three jobs that run in sequence: (1) **lint** — runs `ruff check` on every push, catches formatting and code quality issues instantly. (2) **unit-tests** — runs `pytest tests/unit` — fast gate, fails the PR if any unit test breaks. (3) **integration-tests** — spins up a test PostgreSQL via GitHub Actions services, runs `pytest tests/integration`, verifies the full API works.

**When to use this:** Your source code is hosted on GitHub and your team uses GitHub Actions for CI/CD.

📎 **Reference:** [GitHub Actions Documentation](https://docs.github.com/en/actions)

---

#### Option B: `.azdo/` + `azure-pipelines.yml` (Azure DevOps)

**What it is:** Azure DevOps pipeline configuration. Unlike GitHub, Azure DevOps does **not** enforce a specific folder path — `.azdo/` is a semi-official convention used by Microsoft's own Azure Developer CLI (`azd`) tool and referenced in Microsoft's documentation, but the platform will read templates from any path you reference. The structure inside `.azdo/` is a team-defined convention, not a platform requirement.

**When to use this:** Your source code is hosted in Azure Repos and your team uses Azure Pipelines for CI/CD. This is common in enterprise environments that run on Azure infrastructure.

The `.azdo/` folder contains two subdirectories and the main pipeline file lives at the project root:

---

#### `azure-pipelines.yml` (at project root)

**What it is:** The main entry point for Azure DevOps Pipelines — equivalent to `.github/workflows/ci.yml` in GitHub. This file defines the triggers (which branches activate the pipeline), the stages (build, test, deploy), and references the reusable templates inside `.azdo/templates/`.

**InsightBot example:** Triggers on push to `main` and `develop`. Defines three stages: Build (compile and test), Deploy to Dev (auto-deploy on merge to develop), Deploy to Production (manual approval gate on merge to main). Each stage calls a template from `.azdo/templates/` with environment-specific variables from `.azdo/variables/`.

**Why it matters:** This is the single file Azure DevOps reads to understand your pipeline. It acts as an orchestrator — the actual build/deploy logic lives in the templates it references. Keeping this file thin and delegating to templates is the recommended pattern from Microsoft's pipeline scaling guide. A 500-line monolithic `azure-pipelines.yml` is the Azure DevOps equivalent of a 500-line `main.py` — it means nobody thought about separation of concerns.

📎 **Reference:** [Azure Pipelines — YAML Schema](https://learn.microsoft.com/en-us/azure/devops/pipelines/yaml-schema/)

---

#### `.azdo/templates/`

**What it is:** Reusable YAML pipeline templates, one per deployable component. Each template is a self-contained recipe for building and deploying one specific thing. The main `azure-pipelines.yml` calls these templates with parameters.

**Why there are three files — they map to different Azure compute targets:**

**`backend-container-app.yml`** — Builds and deploys the FastAPI backend as an Azure Container App. Container Apps is Azure's managed container platform. This template handles: build the Docker image → push to Azure Container Registry → deploy to the Container App environment. It exists as its own template because the backend has its own Dockerfile, its own environment variables (database URL, API keys, model config), its own scaling rules (scale on HTTP request count or queue depth), and its own health check endpoint.

**InsightBot example:** Builds the InsightBot API image, pushes to ACR, deploys to the `insightbot-api` Container App. Includes a post-deploy step that runs `make test-integration` against the deployed environment to verify the deployment didn't break anything.

**`frontend-container-app.yml`** — Same concept but for the frontend application (e.g. a React or Next.js app). It is a separate template because the frontend has a completely different build process (`npm run build` vs `pip install`), different Dockerfile, different environment variables (API URL and public keys only — no database credentials or LLM API keys), and different scaling characteristics. Even though both are Container Apps, their build and deploy steps are different enough that combining them into one template would mean ugly conditional branching everywhere.

**InsightBot example:** Builds the InsightBot dashboard UI, deploys to the `insightbot-frontend` Container App with the `API_BASE_URL` variable pointing at the backend's URL.

**`function-app.yml`** — Builds and deploys code to Azure Functions, which is a serverless compute service. This is for lightweight, event-triggered workloads that do not justify running a full container 24/7. Functions have a fundamentally different deployment process (zip deploy, not Docker), different configuration (function app settings, `host.json`), and a different scaling model (scale to zero when idle, pay per execution).

**InsightBot example:** Deploys three functions: (1) a Blob Storage trigger that kicks off document ingestion when new PDFs are uploaded to the `documents` container, (2) a timer trigger that runs the nightly eval suite at 2 AM, (3) a queue trigger that processes background embedding jobs pushed by the API.

**Why three separate templates instead of one big pipeline?**

Each template is independently callable.
📎 **Reference:** [Azure Pipelines — Templates](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/templates) If you only changed backend code, you can run just the backend template — no reason to rebuild and redeploy the frontend or functions. This saves CI/CD minutes and reduces blast radius (a bad frontend deploy doesn't affect the API). Different teams can own different templates without stepping on each other's YAML. And if your organization has multiple projects that all deploy Container Apps, the `backend-container-app.yml` template can be shared via a central templates repository — change it once, every project benefits.

---

#### `.azdo/variables/`

**What it is:** Environment-specific variable groups defined as YAML files. Each file contains the configuration values that change between environments — resource names, subscription IDs, feature flags, scaling limits — but NOT secrets (those go in Azure DevOps Library variable groups linked in the UI, or Azure Key Vault).

**`dev.yml`** — Development environment variables. Typically: smaller instance sizes, debug logging enabled, relaxed rate limits, test API keys.

**`staging.yml`** — Staging/UAT environment. Mirrors production configuration but points at staging resources. Used for final validation before production release.

**`production.yml`** — Production environment. Full instance sizes, error-level logging, strict rate limits, production API keys (referenced from Key Vault, not hardcoded).

**InsightBot example:** `dev.yml` sets `CONTAINER_APP_NAME: insightbot-api-dev`, `LOG_LEVEL: DEBUG`, `MAX_REPLICAS: 2`. `production.yml` sets `CONTAINER_APP_NAME: insightbot-api-prod`, `LOG_LEVEL: ERROR`, `MAX_REPLICAS: 10`. The backend template reads these variables so the same template deploys to any environment — only the variable file changes.

**Why it matters:** Without centralized variable files, environment-specific config ends up hardcoded in pipeline YAML via `if/else` conditions or duplicated across multiple pipeline files. Variable files give you a single source of truth per environment. When you need to change the production replica count, you edit one line in `production.yml` — not hunt through 300 lines of pipeline YAML.

📎 **Reference:** [Azure Pipelines — Variables](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/variables)

---

## Summary: Why Each Folder Exists

| Folder | One-line purpose | Would InsightBot break without it? |
|---|---|---|
| `app/core/` | Config, security, exceptions, constants | Yes — app won't start |
| `app/db/` | Database engine and session management | Yes — no data persistence |
| `app/models/` | Database table definitions (ORM) | Yes — no data structure |
| `app/schemas/` | API request/response contracts (Pydantic) | Yes — no input validation |
| `app/repositories/` | Data access layer between services and DB | Risk — business logic and queries entangled |
| `app/api/v1/` | HTTP routes and dependencies | Yes — no API surface |
| `app/api/v1/endpoints/` | Per-resource route files | Risk — router grows unmanageable |
| `app/services/` | Business logic connecting routes to AI | Yes — routes have no logic |
| `app/agents/` | LLM reasoning units | Yes — no AI capability |
| `app/pipelines/` | End-to-end AI workflows | Yes — no RAG, no ingestion |
| `app/tools/` | Functions agents can call | Yes — agents can't act |
| `app/prompts/` | Versioned prompt templates | Yes — agents have no instructions |
| `app/guardrails/` | Content safety and validation | Risk — PII leaks, hallucinations |
| `app/memory/` | Agent memory and conversation state | Risk — stateful agents lose context |
| `app/retrievers/` | RAG retrieval logic (vector search, reranking) | Risk — retrieval mixed into tools/services |
| `app/middlewares/` | Logging, tracing, rate limiting | Risk — blind to production issues |
| `app/workers/` | Background task processing | Risk — slow API, lost work |
| `alembic/` | Database migrations | Risk — manual schema changes |
| `tests/unit/` | Fast isolated tests | Risk — regressions ship to prod |
| `tests/integration/` | Cross-component tests | Risk — integration failures |
| `tests/e2e/` | Full-flow end-to-end tests | Risk — complete user workflows untested |
| `evals/datasets/` | Golden Q&A pairs and adversarial inputs | Risk — no repeatable eval ground truth |
| `evals/runners/` | Scripts that execute evaluations | Risk — eval runs are ad hoc and inconsistent |
| `evals/results/` | Eval output scores over time | Risk — no quality trend or regression detection |
| `data/raw/` | Original unmodified source data | Convenience — cloud alternative exists |
| `data/processed/` | Chunked, ready-to-embed data | Convenience — cloud alternative exists |
| `data/vectors/` | Local embeddings for RAG development | Convenience — avoids hitting prod vector DB |
| `notebooks/` | Exploration and prototyping | Convenience — not required |
| `scripts/` | CLI utilities | Convenience — can live elsewhere |
| `.dockerignore` | Exclude files from Docker build context | Risk — bloated images, secrets in containers |
| `.pre-commit-config.yaml` | Automated local quality checks before commit | Convenience — CI is the safety net |
| `CONTRIBUTING.md` | Guide for adapting and contributing to the template | Risk — adopters guess instead of follow |
| `.github/workflows/` | CI/CD automation (GitHub) | Risk — no automated quality gate |
| `.azdo/templates/` | Reusable deploy recipes (Azure DevOps) | Risk — no automated quality gate |
| `.azdo/variables/` | Per-environment config (Azure DevOps) | Risk — config scattered in pipeline YAML |
| `azure-pipelines.yml` | Main pipeline orchestrator (Azure DevOps) | Risk — no automated quality gate |

---

## Getting Started

```bash
# 1. Clone this template
git clone <your-repo-url> my-project
cd my-project

# 2. Set up your environment
cp .env.example .env
# Edit .env with your actual values

# 3. Install dependencies
pip install -e ".[dev]"

# 4. Start local services
make docker-up

# 5. Run migrations
make migrate

# 6. Start the dev server
make run

# 7. Open the API docs
# → http://localhost:8000/docs
```

---

## Credits and Sources

This template structure was validated against the following production sources:

- **fastapi-best-practices** (zhanymkanov) — Modular domain-based structure with per-domain schemas, models, and services
- **Cookiecutter Data Science v2** (DrivenData) — 7.6k+ stars, 8+ years in production, staged data directories, Makefile conventions
- **FastAPI LangGraph production template** (wassim249) — 2k stars, includes evals/, observability, structured logging
- **FastAPI official documentation** — Larger Applications guide with routers and sub-applications
- **"Architecting Scalable FastAPI Systems for LLM Applications"** (Medium) — Domain-driven design over technical layering for AI backends
- **"Building LLM apps with FastAPI — best practices"** (Agents Arcade) — Prompt versioning, cost controls, background tasks
- **"Steal This ML Project Structure"** (Medium) — Makefile as discipline signal, staged data directories, entrypoint contracts
- **Towards Data Science MLOps guide** — Project structure with separated pipelines, models, and data versioning
- **Microsoft Learn — Azure Pipelines Templates** — Official documentation on YAML template reuse and scaling
- **Microsoft Community Hub — Practices for Scaling Templates** — Folder-per-type convention, naming standards, template expansion

---

*Template version: 1.2 | Last validated: March 2026*
