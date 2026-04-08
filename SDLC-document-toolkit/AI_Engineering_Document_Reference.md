# AI / LLM Engineering Document Reference

> A companion to the [SDLC Document Reference Guide](SDLC_Document_Reference_Guide.md), covering documents specific to AI and LLM systems. These documents address concerns that traditional SDLC artifacts do not — model provenance, prompt governance, evaluation methodology, AI-specific risk, observability, and dataset integrity.

---

## How to Use This Guide

1. **Use alongside the SDLC guide** — AI projects still need most standard SDLC documents (PRD, HLD, Test Plan, etc.). This guide covers the additional artifacts unique to AI/ML systems.
2. **Pick the documents relevant to your AI maturity** — a prototype using a hosted API needs fewer of these than a fine-tuned model in production.
3. **Same structure** — each entry follows the same format as the SDLC guide: Purpose, Key Components, Owner, Audience, When, Template, References.

---

## AI.1 Model Card

**Purpose:** Describes an AI model's intended use, training data, evaluation results, limitations, and ethical considerations. The model card is the primary accountability artifact for any model you build, fine-tune, or deploy.

**Key Components:**
- Model name, version, and type (foundation, fine-tuned, distilled)
- Intended use cases and out-of-scope uses
- Training data summary (sources, size, date range, known gaps)
- Evaluation results (benchmarks, metrics, comparison to baselines)
- Performance across demographic groups (fairness analysis)
- Known limitations and failure modes
- Ethical considerations and bias risks
- Deployment environment and resource requirements
- Maintenance and update cadence
- Contact and ownership

**Owner:** ML Engineer or AI Lead
**Audience:** Product team, reviewers, compliance, downstream consumers
**When:** Before any model is deployed to production; updated with each retrain or version bump
**Template:** `templates/ai-engineering/MODEL_CARD.md`

**References:**
- Mitchell et al., "Model Cards for Model Reporting" (2019) — the original paper — [arxiv.org/abs/1810.03993](https://arxiv.org/abs/1810.03993)
- Hugging Face Model Card Guide — [huggingface.co/docs/hub/model-cards](https://huggingface.co/docs/hub/model-cards)
- Google Model Cards — [modelcards.withgoogle.com](https://modelcards.withgoogle.com)

---

## AI.2 AI Risk Assessment

**Purpose:** Identifies and evaluates risks specific to AI systems — bias, hallucination, safety, adversarial attacks, and regulatory exposure. Distinct from a general Risk Register (7.3) because it addresses failure modes unique to probabilistic systems.

**Key Components:**
- Risk category (bias/fairness, hallucination, safety, privacy, security, regulatory)
- Risk description and trigger conditions
- Likelihood and impact rating
- Affected populations or use cases
- Detection method (how you would know this risk materialized)
- Mitigation strategy (guardrails, filters, human-in-the-loop)
- Residual risk and acceptance criteria
- Regulatory mapping (EU AI Act risk tier, NIST AI RMF category)
- Review cadence and escalation path

**Owner:** AI Lead or Responsible AI Officer
**Audience:** Leadership, compliance, legal, product team
**When:** Before deployment; revisited quarterly or after significant model changes
**Template:** `templates/ai-engineering/AI_RISK_ASSESSMENT.md`

**References:**
- NIST AI Risk Management Framework (AI RMF 1.0) — [nist.gov/artificial-intelligence/risk-management-framework](https://www.nist.gov/artificial-intelligence/risk-management-framework)
- EU AI Act — risk classification and requirements — [artificialintelligenceact.eu](https://artificialintelligenceact.eu)
- ISO/IEC 23894:2023 — AI Risk Management
- OECD AI Principles — [oecd.ai/en/ai-principles](https://oecd.ai/en/ai-principles)

---

## AI.3 Prompt Management Document

**Purpose:** Governs how prompts are authored, versioned, tested, and deployed. In LLM-based systems, prompts are code — they control behavior, and unmanaged prompt changes cause production regressions.

**Key Components:**
- Prompt inventory (all prompts in the system, by feature/module)
- Prompt template with variable slots
- Version history and changelog per prompt
- Ownership (who can modify each prompt)
- Evaluation criteria (what "good output" looks like for this prompt)
- A/B testing and rollout strategy
- Prompt injection defenses and input sanitization rules
- Model-specific notes (behavior differences across model versions)
- Storage and retrieval mechanism (hardcoded, config, database, prompt registry)

**Owner:** AI Engineer or Prompt Engineer
**Audience:** Developers, QA, product team
**When:** When the system uses LLM prompts; maintained continuously
**Template:** `templates/ai-engineering/PROMPT_MANAGEMENT.md`

**References:**
- Anthropic Prompt Engineering Guide — [docs.anthropic.com/en/docs/build-with-claude/prompt-engineering](https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering)
- OpenAI Prompt Engineering Guide — [platform.openai.com/docs/guides/prompt-engineering](https://platform.openai.com/docs/guides/prompt-engineering)
- OWASP LLM Top 10 — Prompt Injection — [owasp.org/www-project-top-10-for-large-language-model-applications](https://owasp.org/www-project-top-10-for-large-language-model-applications/)

---

## AI.4 Evaluation Framework Document

**Purpose:** Defines how LLM and AI outputs are evaluated — rubrics, benchmarks, human review protocols, and automated scoring. Without a documented eval framework, quality assessments are subjective and unrepeatable.

**Key Components:**
- Evaluation dimensions (accuracy, relevance, safety, format compliance, latency)
- Rubric per dimension (what scores 1 vs. 3 vs. 5)
- Benchmark datasets and expected performance
- Automated eval pipeline (tools, scripts, CI integration)
- Human evaluation protocol (annotator guidelines, inter-rater agreement)
- Regression test suite (golden examples that must always pass)
- Eval cadence (per-commit, per-release, periodic)
- Threshold definitions (what score blocks deployment)
- Comparison methodology (model A vs. model B, prompt v1 vs. v2)

**Owner:** AI Engineer or ML Engineer
**Audience:** AI team, QA, product team
**When:** Before first deployment; updated when prompts, models, or use cases change
**Template:** `templates/ai-engineering/EVAL_FRAMEWORK.md`

**References:**
- Anthropic Evals — [docs.anthropic.com/en/docs/build-with-claude/develop-tests](https://docs.anthropic.com/en/docs/build-with-claude/develop-tests)
- EleutherAI LM Evaluation Harness — [github.com/EleutherAI/lm-evaluation-harness](https://github.com/EleutherAI/lm-evaluation-harness)
- HELM (Holistic Evaluation of Language Models) — [crfm.stanford.edu/helm](https://crfm.stanford.edu/helm/)
- "Evaluating Large Language Models: A Comprehensive Survey" — [arxiv.org/abs/2310.19736](https://arxiv.org/abs/2310.19736)

---

## AI.5 AI Observability Document

**Purpose:** Defines what is monitored, logged, and traced for AI-specific concerns — token usage, latency, cost, output quality drift, and safety violations. Extends the general Monitoring Config (6.5) with AI-specific telemetry.

**Key Components:**
- Metrics collected (tokens in/out, latency p50/p95/p99, cost per call, error rate)
- Quality metrics (output relevance scores, hallucination rate, safety filter triggers)
- Model drift detection (distribution shift in inputs or outputs over time)
- Cost tracking and budget alerts (daily/monthly spend by model, by feature)
- Tracing setup (request → prompt → model call → response → post-processing)
- Logging policy (what is stored, what is redacted for privacy, retention period)
- Dashboard inventory (cost dashboard, quality dashboard, latency dashboard)
- Alerting rules (cost spike, latency degradation, safety violation rate)
- Feedback loop integration (user thumbs up/down → eval pipeline)

**Owner:** AI Engineer or SRE
**Audience:** AI team, DevOps, finance (for cost), compliance (for logging)
**When:** Before production deployment; maintained continuously
**Template:** `templates/ai-engineering/AI_OBSERVABILITY.md`

**References:**
- Langfuse (open-source LLM observability) — [langfuse.com](https://langfuse.com)
- LangSmith — [smith.langchain.com](https://smith.langchain.com)
- "Observability Engineering" by Charity Majors et al. (O'Reilly) — adapted for AI workloads
- OpenTelemetry Semantic Conventions for GenAI — [opentelemetry.io](https://opentelemetry.io)

---

## AI.6 Dataset Documentation

**Purpose:** Documents the provenance, composition, quality, and intended use of datasets used for training, fine-tuning, evaluation, or RAG retrieval. Datasets are a liability if undocumented — bias, licensing, and quality issues hide in undocumented data.

**Key Components:**
- Dataset name, version, and purpose (training / eval / RAG corpus / few-shot examples)
- Source and provenance (where it came from, how it was collected)
- Composition (size, format, schema, label distribution)
- Collection methodology (scraping, annotation, synthetic generation, user data)
- Labeling methodology (annotator guidelines, inter-rater agreement, quality checks)
- Known biases and limitations
- Licensing and usage rights
- PII handling and privacy measures (anonymization, consent)
- Data quality metrics (completeness, consistency, freshness)
- Update and refresh cadence
- Storage location and access controls

**Owner:** Data Engineer or ML Engineer
**Audience:** AI team, compliance, legal
**When:** Before any dataset is used for training or evaluation; updated with each refresh
**Template:** `templates/ai-engineering/DATASET_DOCUMENTATION.md`

**References:**
- Gebru et al., "Datasheets for Datasets" (2021) — the foundational paper — [arxiv.org/abs/1803.09010](https://arxiv.org/abs/1803.09010)
- Hugging Face Dataset Cards — [huggingface.co/docs/hub/datasets-cards](https://huggingface.co/docs/hub/datasets-cards)
- Google Dataset Search — [datasetsearch.research.google.com](https://datasetsearch.research.google.com)

---

## Quick Reference: AI Document Cheat Sheet

| # | Document | Owner | One-Liner | Prototype | Production |
|---|----------|-------|-----------|-----------|------------|
| AI.1 | Model Card | ML Engineer | What the model does and doesn't do | | * |
| AI.2 | AI Risk Assessment | AI Lead | Bias, hallucination, and safety risks | | * |
| AI.3 | Prompt Management | AI Engineer | Prompt versioning, ownership, and testing | * | * |
| AI.4 | Eval Framework | AI Engineer | How AI outputs are scored and gated | * | * |
| AI.5 | AI Observability | AI Engineer / SRE | Token, cost, quality, and drift monitoring | | * |
| AI.6 | Dataset Documentation | Data Engineer | Data provenance, quality, and licensing | | * |

---

*Companion to the [SDLC Document Reference Guide](SDLC_Document_Reference_Guide.md). Last updated: April 2026*
