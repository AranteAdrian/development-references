# Changelog

<!-- Update this file per release, not per commit.
     Add new entries under [Unreleased] as you work.
     When you cut a release, rename [Unreleased] to the version and date,
     then open a fresh [Unreleased] section above it. -->

All notable changes to this template are documented here.
Format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).
Versions follow [Semantic Versioning](https://semver.org/) (MAJOR.MINOR.PATCH).

---

## [Unreleased]

---

## [1.2.0] - 2026-03-20

### Added
- `app/repositories/` — data access layer between services and the database
- `app/retrievers/` — RAG retrieval logic (vector search, reranking)
- `app/memory/` — agent memory and conversation state
- `app/workers/` — background task processing (Celery, async jobs)
- `tests/e2e/` — end-to-end tests for full user-facing workflows
- `evals/` — AI evaluation harness (datasets, runners, results)
- `data/vectors/` — local vector store for RAG development
- `.azdo/` — Azure DevOps pipeline structure (templates + variables)
- `azure-pipelines.yml` — main Azure DevOps pipeline entry point
- `.dockerignore` — excludes data, secrets, and build artifacts from Docker images
- `.pre-commit-config.yaml` — automated pre-commit quality checks (ruff, bandit)
- `CONTRIBUTING.md` — guide for adapting and contributing to the template
- `CHANGELOG.md` — this file

### Changed
- README expanded with full step-by-step walkthrough for every folder (Phases 1–12)
- README summary table updated to cover all new folders and root files
