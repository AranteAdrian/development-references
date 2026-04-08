# Software Engineering Document Reference Guide

> A complete reference of every document type across the Software Development Lifecycle (SDLC). Each entry covers what the document is, why it exists, what it contains, who owns it, and where to learn more. Companion templates are provided in the `/templates` folder.

---

## How to Use This Guide

1. **Identify your phase** — find where you are in the SDLC
2. **Pick the document** — each entry tells you when and why you need it
3. **Use the template** — each document has a ready-to-fill template in `/templates`
4. **Go deeper** — references point you to industry standards and best practices

---

## Phase 1: Discovery + Planning

*These documents define **what** to build and **why**. They translate business needs into actionable requirements and set the project boundaries.*

---

### 1.1 BRD — Business Requirements Document

**Purpose:** Captures the business problem, objectives, and success criteria from the stakeholder's perspective. Answers "why are we building this?"

**Key Components:**
- Business objectives and goals
- Current state / problem statement
- Desired future state
- Scope and boundaries (in-scope / out-of-scope)
- Stakeholders and their expectations
- Success metrics and KPIs
- Constraints (budget, timeline, regulatory)
- Assumptions and dependencies

**Owner:** Business Analyst or Product Owner
**Audience:** Stakeholders, project sponsors, product team
**When:** Before any design or development begins
**Template:** `templates/01-discovery/BRD.md`

**References:**
- IIBA BABOK Guide v3 — Chapter 5: Requirements Analysis and Design Definition — [iiba.org](https://www.iiba.org/business-analysis-certifications/babok-guide/)
- Karl Wiegers, "Software Requirements" (3rd Ed., Microsoft Press) — the gold standard book on requirements engineering
- IEEE 830-1998 (now superseded by ISO/IEC/IEEE 29148:2018) — formal standard for requirements specifications

---

### 1.2 PRD — Product Requirements Document

**Purpose:** Translates the BRD into product-level detail. Describes what the product should do from a user's perspective, including features, user stories, and acceptance criteria.

**Key Components:**
- Product vision and objectives
- Target users / personas
- User stories or use cases with acceptance criteria
- Feature list with priority (MoSCoW or P0/P1/P2)
- User journey maps
- Release criteria / definition of done
- Success metrics
- Out-of-scope items

**Owner:** Product Manager
**Audience:** Engineering, design, QA, stakeholders
**When:** After BRD approval, before design phase
**Template:** `templates/01-discovery/PRD.md`

**References:**
- Marty Cagan, "Inspired: How to Create Tech Products Customers Love" — the definitive product management book
- Atlassian PRD guide — [atlassian.com/agile/product-management/requirements](https://www.atlassian.com/agile/product-management/requirements)
- Silicon Valley Product Group — [svpg.com](https://www.svpg.com/product-requirements-vs-product-spec/)

---

### 1.3 FRD — Functional Requirements Document

**Purpose:** Specifies the exact functional behaviors the system must exhibit. Describes what the system does in response to inputs, including business rules and data validations.

**Key Components:**
- Functional requirement statements (numbered, testable)
- Input/output specifications
- Business rules and logic
- Data validation rules
- Error handling behaviors
- Workflow descriptions
- Interface requirements (UI, API, integrations)
- Traceability to BRD/PRD items

**Owner:** Business Analyst or Systems Analyst
**Audience:** Developers, QA, architects
**When:** During or after requirements gathering
**Template:** `templates/01-discovery/FRD.md`

**References:**
- ISO/IEC/IEEE 29148:2018 — Systems and software engineering — Requirements engineering — [iso.org](https://www.iso.org/standard/72089.html)
- IEEE 830 (legacy but still widely referenced) — Recommended Practice for Software Requirements Specifications
- Alistair Cockburn, "Writing Effective Use Cases" (Addison-Wesley)

---

### 1.4 NFR — Non-Functional Requirements

**Purpose:** Defines the quality attributes the system must meet — how well it performs rather than what it does. Often called "quality requirements" or "-ilities."

**Key Components:**
- Performance (response time, throughput, concurrent users)
- Scalability (horizontal/vertical, load projections)
- Availability and reliability (uptime SLA, MTTR, MTBF)
- Security (authentication, authorization, encryption, compliance)
- Maintainability (code quality, modularity, documentation)
- Usability (accessibility, localization, UX standards)
- Portability (platforms, browsers, devices)
- Disaster recovery (RPO, RTO, backup strategy)

**Owner:** Architect or Technical Lead
**Audience:** Engineering, DevOps, security, QA
**When:** Alongside FRD, before architecture decisions
**Template:** `templates/01-discovery/NFR.md`

**References:**
- ISO/IEC 25010:2011 — Systems and software quality models (defines the "-ilities") — [iso.org](https://www.iso.org/standard/35733.html)
- Bass, Clements, Kazman, "Software Architecture in Practice" (4th Ed., Addison-Wesley) — Chapter 4: Quality Attributes
- Google SRE Book — [sre.google/sre-book/](https://sre.google/sre-book/) — practical take on reliability and performance requirements

---

### 1.5 Project Charter

**Purpose:** Formally authorizes the project. Provides a high-level overview of scope, objectives, stakeholders, timeline, and budget.

**Key Components:**
- Project name and description
- Business case / justification
- Objectives and deliverables
- High-level scope
- Key milestones and timeline
- Budget estimate
- Stakeholders and roles
- Risks and assumptions
- Approval signatures

**Owner:** Project Manager
**Audience:** Sponsors, executives, project team
**When:** Project initiation, before any work begins
**Template:** `templates/01-discovery/PROJECT_CHARTER.md`

**References:**
- PMBOK Guide (7th Ed., PMI) — Section 4.1: Develop Project Charter — [pmi.org](https://www.pmi.org/pmbok-guide-standards)
- PRINCE2 — Project Brief and Project Initiation Document (PID) — [axelos.com](https://www.axelos.com/certifications/prince2)

---

### 1.6 SOW — Statement of Work

**Purpose:** Defines the specific work to be performed, deliverables, timelines, and payment terms. Used in vendor/client engagements.

**Key Components:**
- Scope of work (detailed task descriptions)
- Deliverables with acceptance criteria
- Timeline and milestones
- Payment schedule and terms
- Roles and responsibilities
- Assumptions and exclusions
- Change management process
- Termination clauses

**Owner:** Project Manager or Account Manager
**Audience:** Client, vendor, legal, procurement
**When:** Before contract signing
**Template:** `templates/01-discovery/SOW.md`

**References:**
- FAR (Federal Acquisition Regulation) Part 37 — formal SOW standards for government contracts — [acquisition.gov](https://www.acquisition.gov/far/part-37)
- PMI Practice Standard for Work Breakdown Structures (3rd Ed.)

---

### 1.7 RFP / RFQ / RFI

**Purpose:** Formal documents used to solicit proposals, quotes, or information from vendors.

| Document | Full Name | Purpose |
|----------|-----------|---------|
| RFI | Request for Information | Gather general info about vendor capabilities |
| RFQ | Request for Quotation | Get pricing for a well-defined scope |
| RFP | Request for Proposal | Get detailed proposals including approach, timeline, cost |

**Key Components (RFP):**
- Company background and project context
- Scope of work and requirements
- Evaluation criteria and weighting
- Submission format and deadline
- Questions / clarification process
- Contract terms and conditions

**Owner:** Procurement or Project Manager
**Audience:** External vendors
**When:** When evaluating external solutions or partners
**Template:** `templates/01-discovery/RFP.md`

**References:**
- CIPS (Chartered Institute of Procurement & Supply) — RFx best practices — [cips.org](https://www.cips.org)
- Gartner — "How to Write an Effective RFP for Technology"

---

### 1.8 Stakeholder Analysis

**Purpose:** Identifies all stakeholders, their interests, influence level, and communication needs.

**Key Components:**
- Stakeholder register (name, role, organization)
- Interest and influence matrix (power/interest grid)
- Communication preferences and frequency
- Expectations and concerns per stakeholder
- RACI assignment for key decisions

**Owner:** Project Manager
**Audience:** Project team, leadership
**When:** Project initiation, updated throughout
**Template:** `templates/01-discovery/STAKEHOLDER_ANALYSIS.md`

**References:**
- PMBOK Guide — Section 13: Project Stakeholder Management
- Mendelow's Power/Interest Matrix — standard stakeholder classification tool

---

### 1.9 Feasibility Study

**Purpose:** Evaluates whether the project is technically, financially, and operationally viable before committing resources.

**Key Components:**
- Technical feasibility (can we build it?)
- Economic feasibility (cost-benefit analysis, ROI)
- Operational feasibility (will users adopt it?)
- Schedule feasibility (can we deliver in time?)
- Legal/regulatory feasibility
- Recommendation (go / no-go / conditional)

**Owner:** Business Analyst or Architect
**Audience:** Sponsors, leadership, project team
**When:** Before project charter approval
**Template:** `templates/01-discovery/FEASIBILITY_STUDY.md`

**References:**
- PMBOK Guide — Section 1.2.6: Business Documents
- ISO 21500:2012 — Guidance on project management

---

## Phase 2: Design + Architecture

*These documents define **how** to build the system. They bridge requirements and code by establishing the technical blueprint.*

---

### 2.1 HLD — High-Level Design

**Purpose:** Provides the big-picture architecture — system components, their interactions, technology choices, and data flow.

**Key Components:**
- System architecture diagram
- Component / service breakdown
- Technology stack and rationale
- Integration points (APIs, queues, databases)
- Data flow overview
- Deployment topology (cloud, on-prem, hybrid)
- Security architecture overview
- Key design decisions and trade-offs

**Owner:** Solution Architect
**Audience:** Engineering leads, stakeholders, DevOps
**When:** After requirements are stable, before detailed design
**Template:** `templates/02-design/HLD.md`

**References:**
- ISO/IEC/IEEE 42010:2011 — Architecture description — [iso.org](https://www.iso.org/standard/50508.html)
- Simon Brown, "Software Architecture for Developers" — C4 model for diagramming — [c4model.com](https://c4model.com)
- Martin Fowler, "Patterns of Enterprise Application Architecture" (Addison-Wesley)

---

### 2.2 LLD — Low-Level Design

**Purpose:** Drills into each component from the HLD with implementation-level detail.

**Key Components:**
- Detailed class / module diagrams
- Sequence diagrams for key flows
- Database schema (tables, relationships, indexes)
- API endpoint specifications (request/response)
- Algorithm descriptions and pseudocode
- Error handling strategies per component
- State machine diagrams (where applicable)
- Mapping to HLD components

**Owner:** Technical Lead or Senior Developer
**Audience:** Developers, code reviewers
**When:** After HLD approval, before coding starts
**Template:** `templates/02-design/LLD.md`

**References:**
- UML 2.5 Specification — [omg.org/spec/UML](https://www.omg.org/spec/UML)
- Craig Larman, "Applying UML and Patterns" (3rd Ed., Prentice Hall)

---

### 2.3 SAD — Solution Architecture Document

**Purpose:** The comprehensive technical blueprint combining HLD and LLD into a single reference. Common in enterprise environments where a formal architecture review board (ARB) must approve designs.

**Key Components:**
- Architecture overview and principles
- Context diagram (system in its environment)
- Component architecture (logical and physical)
- Integration architecture
- Data architecture (models, flows, storage)
- Security architecture
- Infrastructure and deployment architecture
- Performance and scalability design
- Disaster recovery design
- Architecture decision records (ADRs)

**Owner:** Solution Architect or Enterprise Architect
**Audience:** ARB, engineering, DevOps, security
**When:** Before development, reviewed periodically
**Template:** `templates/02-design/SAD.md`

> **Naming note:** Some organizations call this document an "SDD" (Solution Design Document) or "TAD" (Technical Architecture Document). In this guide, "SAD" refers to the architecture-level blueprint (this entry), while "SDD" (3.1) refers to the developer-facing implementation spec. If your organization uses different terminology, map accordingly.

**References:**
- TOGAF (The Open Group Architecture Framework) — [opengroup.org/togaf](https://www.opengroup.org/togaf)
- Arc42 — lean architecture documentation template — [arc42.org](https://arc42.org)
- IEEE 1471-2000 (now ISO/IEC/IEEE 42010) — Recommended Practice for Architectural Description

---

### 2.4 ERD — Entity Relationship Diagram

**Purpose:** Visual representation of the data model showing entities, attributes, and relationships.

**Key Components:**
- Entities with attributes and data types
- Primary keys and foreign keys
- Relationship types (1:1, 1:N, M:N)
- Cardinality and optionality notation
- Indexes and constraints
- Normalization notes

**Owner:** Database Designer or Backend Developer
**Audience:** Developers, DBAs, data engineers
**When:** During LLD, before database implementation
**Template:** `templates/02-design/ERD.md`

**References:**
- Peter Chen's original ER model paper (1976) — foundational theory
- Mermaid.js erDiagram syntax — [mermaid.js.org](https://mermaid.js.org/syntax/entityRelationshipDiagram.html)
- "Database Design for Mere Mortals" by Michael Hernandez (Addison-Wesley)

---

### 2.5 API Specification

**Purpose:** Defines the contract for APIs — endpoints, methods, request/response schemas, authentication, and error codes.

**Key Components:**
- Base URL and versioning strategy
- Endpoints with HTTP methods
- Request parameters, headers, and body schemas
- Response schemas with status codes
- Authentication / authorization requirements
- Rate limiting and pagination
- Error response format
- Example requests and responses

**Format:** OpenAPI/Swagger (YAML/JSON), GraphQL schema, or AsyncAPI
**Owner:** Backend Lead or API Designer
**Audience:** Frontend devs, mobile devs, integration partners
**When:** During design, maintained throughout development
**Template:** `templates/02-design/API_SPEC.md`

**References:**
- OpenAPI Specification 3.1 — [spec.openapis.org](https://spec.openapis.org/oas/latest.html)
- Swagger Editor — [editor.swagger.io](https://editor.swagger.io)
- AsyncAPI (for event-driven APIs) — [asyncapi.com](https://www.asyncapi.com)
- Google API Design Guide — [cloud.google.com/apis/design](https://cloud.google.com/apis/design)

---

### 2.6 DFD — Data Flow Diagram

**Purpose:** Shows how data moves through the system — from external sources through processes to data stores.

**Key Components:**
- External entities (data sources and sinks)
- Processes (transformations)
- Data stores
- Data flows (arrows with labels)
- Levels: Context (L0), System (L1), Detailed (L2+)

**Owner:** Systems Analyst or Architect
**Audience:** Developers, data engineers, stakeholders
**When:** During design phase
**Template:** `templates/02-design/DFD.md`

**References:**
- Tom DeMarco, "Structured Analysis and System Specification" — original DFD methodology
- Yourdon and Coad notation — standard DFD symbols

---

### 2.7 Wireframes / UI Mockups

**Purpose:** Visual representations of the user interface at varying fidelity — from rough sketches to pixel-perfect designs.

**Key Components:**
- Screen layouts with component placement
- Navigation flow between screens
- Interactive states (hover, active, disabled, error)
- Responsive breakpoints
- Annotation of behaviors and interactions
- Design system / component library references

**Owner:** UX/UI Designer
**Audience:** Product, engineering, stakeholders
**When:** After PRD, before frontend development
**Template:** `templates/02-design/WIREFRAME_SPEC.md`

**References:**
- Nielsen Norman Group — [nngroup.com](https://www.nngroup.com/articles/wireflows/)
- Figma best practices — [figma.com/best-practices](https://www.figma.com/best-practices/)
- Steve Krug, "Don't Make Me Think" (New Riders)

---

### 2.8 ADR — Architecture Decision Record

> **Cross-cutting note:** ADRs are listed under Phase 2 because architecture decisions begin during design, but they are created throughout the entire lifecycle — during pre-design spikes, development trade-offs, operational changes, and technology migrations. Treat ADRs as a living, cross-cutting practice, not a one-time design artifact.

**Purpose:** Captures a single architecture decision — the context, options considered, decision made, and consequences. Creates an immutable decision log over time.

**Key Components:**
- Title (short descriptive name)
- Status (proposed / accepted / deprecated / superseded)
- Context (why this decision is needed)
- Options considered (with pros and cons)
- Decision (which option was chosen)
- Consequences (trade-offs, what changes, what's accepted)
- Date and participants

**Owner:** Architect or Tech Lead
**Audience:** Engineering team, future developers
**When:** Whenever a significant technical decision is made — from pre-design through operations
**Template:** `templates/02-design/ADR.md`

**References:**
- Michael Nygard's ADR format — [cognitect.com/blog/2011/11/15/documenting-architecture-decisions](https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions)
- adr-tools (CLI for managing ADRs) — [github.com/npryce/adr-tools](https://github.com/npryce/adr-tools)
- MADR (Markdown ADR) format — [adr.github.io/madr](https://adr.github.io/madr/)

---

### 2.9 Threat Model

**Purpose:** Systematically identifies security threats, attack surfaces, and trust boundaries during design — before code is written. Distinct from pen testing (4.6), which evaluates the built system; threat modeling evaluates design decisions.

**Key Components:**
- System decomposition (components, data flows, trust boundaries)
- Threat identification using STRIDE (Spoofing, Tampering, Repudiation, Information Disclosure, Denial of Service, Elevation of Privilege)
- Attack surface mapping
- Trust boundary diagrams
- Threat severity rating (DREAD or risk matrix)
- Mitigations per threat (design controls, not code fixes)
- Residual risk acceptance
- Review cadence and triggers for re-assessment

**Owner:** Security Lead or Architect
**Audience:** Architects, developers, security team, compliance
**When:** During design phase, revisited when architecture changes significantly
**Template:** `templates/02-design/THREAT_MODEL.md`

**References:**
- Adam Shostack, "Threat Modeling: Designing for Security" (Wiley) — the definitive book
- OWASP Threat Modeling — [owasp.org/www-community/Threat_Modeling](https://owasp.org/www-community/Threat_Modeling)
- Microsoft STRIDE model — [microsoft.com/en-us/securityengineering/sdl/threatmodeling](https://www.microsoft.com/en-us/securityengineering/sdl/threatmodeling)
- NIST SP 800-154 — Guide to Data-Centric System Threat Modeling

---

## Phase 3: Development + Build

*These documents support the actual coding process — standards, setup guides, and development workflows.*

---

### 3.1 SDD — Software Design Document

**Purpose:** The developer-facing technical spec that bridges the LLD and actual code. Details how a specific feature or module should be implemented.

**Key Components:**
- Feature / module overview
- Technical approach and design rationale
- Component interactions and dependencies
- Data models and schemas
- API contracts (if not covered by API spec)
- Algorithm details
- Configuration and feature flags
- Testing strategy for this feature
- Known limitations and future considerations

**Owner:** Developer or Tech Lead
**Audience:** Developers, code reviewers
**When:** Before implementing a complex feature
**Template:** `templates/03-development/SDD.md`

> **Naming note:** "SDD" is used inconsistently across the industry. Some organizations use "SDD" to mean the architecture-level document covered by SAD (2.3) in this guide. Here, SDD refers specifically to the developer-facing implementation spec for a feature or module — closer to a Google Design Doc than a TOGAF architecture artifact.

**References:**
- IEEE 1016-2009 — Software Design Descriptions — [ieee.org](https://standards.ieee.org/standard/1016-2009.html)
- Google Design Docs — [industrialempathy.com/posts/design-docs-at-google](https://www.industrialempathy.com/posts/design-docs-at-google/)

---

### 3.2 Coding Standards / Style Guide

**Purpose:** Defines code formatting, naming conventions, patterns, and anti-patterns that the team must follow for consistency and maintainability.

**Key Components:**
- Naming conventions (variables, functions, classes, files)
- Formatting rules (indentation, line length, imports)
- Language-specific patterns and anti-patterns
- Error handling conventions
- Logging standards
- Comment and docstring requirements
- Git commit message format
- Linter and formatter configuration

**Owner:** Tech Lead
**Audience:** All developers
**When:** Established at project start, enforced continuously
**Template:** `templates/03-development/CODING_STANDARDS.md`

**References:**
- Google Style Guides — [google.github.io/styleguide](https://google.github.io/styleguide/)
- Airbnb JavaScript Style Guide — [github.com/airbnb/javascript](https://github.com/airbnb/javascript)
- PEP 8 (Python) — [peps.python.org/pep-0008](https://peps.python.org/pep-0008/)
- "Clean Code" by Robert C. Martin (Prentice Hall)

---

### 3.3 README / Setup Guide

**Purpose:** The first document any new developer reads. Covers how to set up the project locally, run it, and contribute.

**Key Components:**
- Project description and purpose
- Prerequisites (runtime, tools, accounts)
- Installation / setup steps
- Environment variable configuration
- How to run locally (dev mode)
- How to run tests
- Project structure overview
- Contributing guidelines
- License

**Owner:** Tech Lead or initial developer
**Audience:** All developers, open-source contributors
**When:** Created at project start, maintained continuously
**Template:** `templates/03-development/README_TEMPLATE.md`

**References:**
- Make a README — [makeareadme.com](https://www.makeareadme.com)
- GitHub's guide to READMEs — [docs.github.com](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/about-readmes)

---

### 3.4 Database Schema / Migration Doc

**Purpose:** Documents the database structure, migration history, and data management procedures.

**Key Components:**
- Current schema (tables, columns, types, constraints)
- Migration history and versioning approach
- Seed data / initial data requirements
- Backup and restore procedures
- Index strategy and query optimization notes
- Data retention and archival policies

**Owner:** Backend Developer or DBA
**Audience:** Developers, DevOps, data engineers
**When:** During development, updated with each migration
**Template:** `templates/03-development/DB_SCHEMA.md`

**References:**
- Alembic (Python) — [alembic.sqlalchemy.org](https://alembic.sqlalchemy.org)
- Flyway — [flywaydb.org](https://flywaydb.org)
- "SQL Antipatterns" by Bill Karwin (Pragmatic Bookshelf)

---

### 3.5 Branching Strategy Doc

**Purpose:** Defines how the team uses Git branches — naming, merging, release workflows, and PR conventions.

**Key Components:**
- Branch naming convention
- Branch types (main, develop, feature, release, hotfix)
- Merge strategy (merge commits, squash, rebase)
- PR review requirements
- CI/CD trigger rules per branch
- Release tagging convention
- Hotfix workflow

**Owner:** Tech Lead or DevOps
**Audience:** All developers
**When:** Project start
**Template:** `templates/03-development/BRANCHING_STRATEGY.md`

**References:**
- GitFlow — [nvie.com/posts/a-successful-git-branching-model](https://nvie.com/posts/a-successful-git-branching-model/)
- GitHub Flow — [docs.github.com/en/get-started/using-github/github-flow](https://docs.github.com/en/get-started/using-github/github-flow)
- Trunk-Based Development — [trunkbaseddevelopment.com](https://trunkbaseddevelopment.com)

---

### 3.6 Configuration Management Doc

**Purpose:** Documents all configuration settings, environment variables, feature flags, and how they differ across environments.

**Key Components:**
- Environment list (local, dev, staging, prod)
- Environment variables per environment
- Feature flags and their states
- Secrets management approach
- Configuration file locations
- How to add/change configuration

**Owner:** DevOps or Backend Developer
**Audience:** Developers, DevOps
**When:** During development, maintained continuously
**Template:** `templates/03-development/CONFIG_MANAGEMENT.md`

**References:**
- 12-Factor App — Factor III: Config — [12factor.net/config](https://12factor.net/config)
- HashiCorp Vault — [vaultproject.io](https://www.vaultproject.io)

---

## Phase 4: Testing + QA

*These documents validate that what was built matches what was specified and meets quality standards.*

---

### 4.1 Test Plan

**Purpose:** The master document for the testing effort. Defines what will be tested, how, by whom, and the pass/fail criteria.

**Key Components:**
- Test objectives and scope
- Test strategy (types of testing to perform)
- Test environment requirements
- Entry and exit criteria
- Test schedule and milestones
- Roles and responsibilities
- Risk assessment and mitigation
- Defect management process
- Tools and infrastructure

**Owner:** QA Lead
**Audience:** QA team, PM, developers
**When:** After requirements are finalized, before testing begins
**Template:** `templates/04-testing/TEST_PLAN.md`

**References:**
- IEEE 829-2008 — Standard for Software Test Documentation — [ieee.org](https://standards.ieee.org/standard/829-2008.html)
- ISTQB Foundation Level Syllabus — [istqb.org](https://www.istqb.org)
- "Lessons Learned in Software Testing" by Kaner, Bach, Pettichord (Wiley)

---

### 4.2 Test Cases / Test Suite

**Purpose:** Individual test scenarios with steps, inputs, expected results, and pass/fail criteria. Grouped into suites by feature or area.

**Key Components:**
- Test case ID and title
- Preconditions
- Test steps (numbered)
- Test data / inputs
- Expected result per step
- Actual result (filled during execution)
- Pass / fail status
- Priority and severity
- Traceability to requirements

**Owner:** QA Engineer
**Audience:** QA team, developers
**When:** Written before test execution, updated during
**Template:** `templates/04-testing/TEST_CASES.md`

**References:**
- ISTQB — Test case design techniques (equivalence partitioning, boundary value, decision table)
- "Agile Testing" by Lisa Crispin and Janet Gregory (Addison-Wesley)

---

### 4.3 UAT — User Acceptance Testing

**Purpose:** Validates that the system meets business requirements from the end user's perspective. The final gate before go-live.

**Key Components:**
- UAT objectives and scope
- UAT scenarios (business process-based)
- Acceptance criteria per scenario
- Test data requirements
- UAT environment details
- Participant list and roles
- Sign-off process and criteria
- Issue escalation path
- UAT summary and sign-off form

**Owner:** Business Analyst or Product Owner
**Audience:** Business users, stakeholders, QA
**When:** After system testing passes, before production release
**Template:** `templates/04-testing/UAT.md`

**References:**
- IIBA BABOK — Chapter 6: Solution Evaluation
- "User Acceptance Testing" by Brian Hambling and Pauline van Goethem (BCS)

---

### 4.4 Bug / Defect Report

**Purpose:** Standardized format for reporting, tracking, and resolving software defects.

**Key Components:**
- Bug ID and title
- Environment (browser, OS, device, environment)
- Steps to reproduce
- Expected vs. actual behavior
- Severity (critical / major / minor / cosmetic)
- Priority (P0–P3)
- Screenshots or recordings
- Assigned to / status
- Root cause (filled on resolution)
- Fix verification

**Owner:** QA Engineer (reports), Developer (fixes)
**Audience:** QA, developers, PM
**When:** Whenever a defect is found
**Template:** `templates/04-testing/BUG_REPORT.md`

**References:**
- "How to Report Bugs Effectively" by Simon Tatham — [chiark.greenend.org.uk](https://www.chiark.greenend.org.uk/~sgtatham/bugs.html)
- Atlassian Jira bug tracking best practices

---

### 4.5 Performance Test Report

**Purpose:** Documents the results of load, stress, and performance testing including benchmarks, bottlenecks, and recommendations.

**Key Components:**
- Test objectives and success criteria
- Test environment and configuration
- Test scenarios (load profile, user concurrency, duration)
- Results: response times, throughput, error rates
- Resource utilization (CPU, memory, disk, network)
- Bottleneck analysis
- Comparison to NFRs / SLAs
- Recommendations and next steps
- Raw data and graphs

**Owner:** Performance Engineer or QA Lead
**Audience:** Architects, DevOps, PM
**When:** Before production release
**Template:** `templates/04-testing/PERFORMANCE_TEST_REPORT.md`

**References:**
- "The Art of Application Performance Testing" by Ian Molyneaux (O'Reilly)
- Apache JMeter — [jmeter.apache.org](https://jmeter.apache.org)
- k6 load testing — [k6.io](https://k6.io)
- Locust — [locust.io](https://locust.io)

---

### 4.6 Security Assessment / Pen Test Report

**Purpose:** Documents findings from security testing — vulnerabilities, risk levels, and remediation recommendations.

**Key Components:**
- Assessment scope and methodology
- Tools used
- Findings with severity rating (CVSS)
- Evidence (screenshots, payloads, logs)
- Risk assessment per finding
- Remediation recommendations with priority
- Retest plan and verification
- Executive summary for leadership

**Owner:** Security Engineer or external pen tester
**Audience:** Developers, architects, CISO, leadership
**When:** Before production release, periodically after
**Template:** `templates/04-testing/SECURITY_ASSESSMENT.md`

**References:**
- OWASP Testing Guide — [owasp.org/www-project-web-security-testing-guide](https://owasp.org/www-project-web-security-testing-guide/)
- OWASP Top 10 — [owasp.org/www-project-top-ten](https://owasp.org/www-project-top-ten/)
- NIST SP 800-115 — Technical Guide to Information Security Testing
- CVSS v3.1 Calculator — [first.org/cvss/calculator/3.1](https://www.first.org/cvss/calculator/3.1)

---

### 4.7 RTM — Requirements Traceability Matrix

> **Cross-cutting note:** The RTM is listed under Phase 4 because it is most actively used during testing, but it is first created during Phase 1 (Requirements) and populated progressively through design and development. By testing time, the RTM should already have requirement-to-design and requirement-to-code mappings — testing adds the final test-case and defect columns.

**Purpose:** Maps every requirement to its corresponding design element, code, test case, and defect. Ensures nothing is missed and provides audit trail.

**Key Components:**
- Requirement ID → Design reference → Code module → Test case ID → Defect ID
- Forward traceability (requirement → test)
- Backward traceability (test → requirement)
- Coverage status (covered / partially / not covered)
- Priority and status per requirement

**Owner:** QA Lead or Business Analyst
**Audience:** QA, PM, auditors
**When:** Created during requirements (Phase 1), populated through design and development, completed during testing
**Template:** `templates/04-testing/RTM.md`

**References:**
- IEEE 830 / ISO 29148 — traceability requirements
- CMMI — Requirements Management (REQM) process area

---

## Phase 5: Deployment + Release

*These documents get the system safely into production and ensure you can recover if something goes wrong.*

---

### 5.1 Deployment Plan / Runbook

**Purpose:** Step-by-step instructions for deploying the system to an environment. Eliminates guesswork and ensures repeatable deployments.

**Key Components:**
- Pre-deployment checklist
- Deployment steps (numbered, in order)
- Environment-specific configurations
- Database migration steps
- Smoke test / verification steps
- Rollback procedure
- Notification plan (who to notify, when)
- Post-deployment verification
- Contact list for escalation

**Owner:** DevOps or Release Manager
**Audience:** DevOps, developers, on-call team
**When:** Before each release
**Template:** `templates/05-deployment/DEPLOYMENT_PLAN.md`

**References:**
- Google SRE Book — Chapter 8: Release Engineering — [sre.google](https://sre.google/sre-book/release-engineering/)
- "Continuous Delivery" by Jez Humble and David Farley (Addison-Wesley)

---

### 5.2 Release Notes

**Purpose:** Communicates what changed in a release — new features, bug fixes, known issues, and breaking changes. Written for both internal teams and end users.

**Key Components:**
- Release version and date
- New features with descriptions
- Bug fixes with ticket references
- Known issues and workarounds
- Breaking changes and migration steps
- Deprecation notices
- Contributors / acknowledgments

**Owner:** Product Manager or Release Manager
**Audience:** Users, stakeholders, support team, developers
**When:** With every release
**Template:** `templates/05-deployment/RELEASE_NOTES.md`

**References:**
- Keep a Changelog — [keepachangelog.com](https://keepachangelog.com)
- Semantic Versioning — [semver.org](https://semver.org)

---

### 5.3 Rollback Plan

**Purpose:** Defines how to revert to the previous working state if a deployment fails or causes critical issues.

**Key Components:**
- Rollback trigger criteria (what constitutes a failed deploy)
- Decision authority (who approves rollback)
- Rollback steps (reverse of deployment)
- Database rollback / migration reversal
- Cache/CDN invalidation steps
- Verification after rollback
- Communication plan (status page, Slack, email)
- Post-mortem trigger

**Owner:** DevOps or Release Manager
**Audience:** DevOps, on-call team, PM
**When:** Created alongside deployment plan
**Template:** `templates/05-deployment/ROLLBACK_PLAN.md`

**References:**
- Google SRE Book — Chapter 14: Managing Incidents
- AWS Well-Architected Framework — Reliability Pillar — [docs.aws.amazon.com](https://docs.aws.amazon.com/wellarchitected/latest/reliability-pillar/)

---

### 5.4 CI/CD Pipeline Doc

**Purpose:** Documents the continuous integration and deployment pipeline — stages, triggers, gates, and configuration.

**Key Components:**
- Pipeline architecture diagram
- Stages (build, test, security scan, deploy)
- Trigger conditions per stage
- Environment promotion flow (dev → staging → prod)
- Quality gates (test coverage, linting, security)
- Secrets and credentials management
- Artifact management
- Monitoring and alerting for pipeline failures

**Owner:** DevOps Engineer
**Audience:** Developers, DevOps, QA
**When:** Project setup, updated as pipeline evolves
**Template:** `templates/05-deployment/CICD_PIPELINE.md`

**References:**
- "Continuous Delivery" by Jez Humble and David Farley
- GitHub Actions docs — [docs.github.com/en/actions](https://docs.github.com/en/actions)
- Azure DevOps Pipelines — [learn.microsoft.com](https://learn.microsoft.com/en-us/azure/devops/pipelines/)

---

### 5.5 Change Request (CR)

**Purpose:** Formal request to modify the system — tracks the what, why, impact, and approval for any change to production.

**Key Components:**
- Change request ID and title
- Description of change
- Reason / justification
- Impact assessment (systems, users, data)
- Risk assessment
- Implementation plan
- Testing plan
- Rollback plan
- Approvals (CAB — Change Advisory Board)
- Schedule

**Owner:** Developer or PM (requests), CAB (approves)
**Audience:** CAB, DevOps, PM, stakeholders
**When:** Before any production change
**Template:** `templates/05-deployment/CHANGE_REQUEST.md`

**References:**
- ITIL 4 — Change Enablement practice — [axelos.com](https://www.axelos.com/certifications/itil-service-management)
- ISO/IEC 20000 — IT Service Management

---

### 5.6 Go-Live Checklist

**Purpose:** A comprehensive checklist of everything that must be verified before and during a production launch.

**Key Components:**
- Infrastructure readiness
- Database migrations applied
- Environment variables configured
- SSL certificates valid
- DNS configured
- Monitoring and alerting active
- Backup verified
- Rollback plan reviewed
- Support team briefed
- Communication sent (status page, customers)
- Smoke tests passed
- Sign-off obtained

**Owner:** Release Manager or PM
**Audience:** Entire project team
**When:** Immediately before go-live
**Template:** `templates/05-deployment/GO_LIVE_CHECKLIST.md`

**References:**
- "The Checklist Manifesto" by Atul Gawande — the case for checklists in complex systems
- Launch Darkly feature flag best practices — [launchdarkly.com](https://launchdarkly.com)

---

## Phase 6: Operations + Maintenance

*These documents keep the system running, help respond to incidents, and guide ongoing maintenance.*

---

### 6.1 SOP — Standard Operating Procedure

**Purpose:** Step-by-step instructions for routine operational tasks — things the ops team does regularly.

**Key Components:**
- Procedure name and purpose
- Scope (which systems, which environments)
- Prerequisites
- Step-by-step instructions
- Expected outcomes per step
- Troubleshooting / what-if scenarios
- Frequency and schedule
- Responsible role

**Owner:** DevOps or Operations Lead
**Audience:** Operations team, on-call engineers
**When:** When a task becomes routine
**Template:** `templates/06-operations/SOP.md`

**References:**
- ITIL 4 — Service Operation practices
- "The Phoenix Project" by Gene Kim, Kevin Behr, George Spafford (IT Revolution)

---

### 6.2 Runbook / Playbook

**Purpose:** Detailed instructions for responding to specific operational scenarios — alerts, failures, scaling events. More specific than an SOP.

**Key Components:**
- Trigger (what alert or event activates this)
- Severity assessment
- Diagnostic steps
- Resolution steps
- Escalation path
- Communication template
- Post-resolution verification
- Automation opportunities

**Owner:** DevOps or SRE
**Audience:** On-call engineers
**When:** When a recurring operational scenario is identified
**Template:** `templates/06-operations/RUNBOOK.md`

**References:**
- Google SRE Book — Chapter 11: Being On-Call — [sre.google](https://sre.google/sre-book/being-on-call/)
- PagerDuty Incident Response Guide — [response.pagerduty.com](https://response.pagerduty.com)

---

### 6.3 Incident Response Plan

**Purpose:** Defines how the team detects, responds to, communicates about, and recovers from production incidents.

**Key Components:**
- Incident severity definitions (SEV1–SEV4)
- Detection mechanisms (monitoring, alerts, user reports)
- Response roles (incident commander, comms lead, tech lead)
- Escalation matrix
- Communication templates (internal, external, status page)
- Resolution workflow
- Post-incident review process
- Incident log template

**Owner:** SRE Lead or Engineering Manager
**Audience:** All engineering, support, leadership
**When:** Before going to production
**Template:** `templates/06-operations/INCIDENT_RESPONSE.md`

**References:**
- Google SRE Book — Chapter 14: Managing Incidents
- PagerDuty Incident Response — [response.pagerduty.com](https://response.pagerduty.com)
- NIST SP 800-61 — Computer Security Incident Handling Guide

---

### 6.4 SLA / SLO Documentation

**Purpose:** Defines the service level agreements (external commitments) and service level objectives (internal targets) for system reliability.

**Key Components:**
- SLIs (Service Level Indicators) — what you measure
- SLOs (Service Level Objectives) — what you target
- SLAs (Service Level Agreements) — what you promise
- Error budget policy
- Measurement methodology
- Reporting cadence
- Consequences of breach
- Review and revision process

**Owner:** SRE or Product Manager
**Audience:** Engineering, leadership, customers
**When:** Before go-live, reviewed quarterly
**Template:** `templates/06-operations/SLA_SLO.md`

**References:**
- Google SRE Book — Chapter 4: Service Level Objectives — [sre.google](https://sre.google/sre-book/service-level-objectives/)
- Google SRE Workbook — Implementing SLOs — [sre.google/workbook](https://sre.google/workbook/implementing-slos/)
- "Implementing Service Level Objectives" by Alex Hidalgo (O'Reilly)

---

### 6.5 Monitoring + Alerting Config Doc

**Purpose:** Documents what is monitored, alert thresholds, notification channels, and dashboard locations.

**Key Components:**
- Metrics collected (system, application, business)
- Alert rules with thresholds and severity
- Notification channels (Slack, PagerDuty, email)
- On-call rotation schedule
- Dashboard URLs and what each shows
- Log aggregation setup
- Trace sampling configuration
- Alert tuning history (to reduce noise)

**Owner:** DevOps or SRE
**Audience:** On-call engineers, DevOps
**When:** During deployment setup, maintained continuously
**Template:** `templates/06-operations/MONITORING.md`

**References:**
- Google SRE Book — Chapter 6: Monitoring Distributed Systems
- "Observability Engineering" by Charity Majors, Liz Fong-Jones, George Miranda (O'Reilly)
- Datadog monitoring best practices — [datadoghq.com](https://www.datadoghq.com/blog/monitoring-best-practices/)

---

### 6.6 Post-Incident Review (PIR)

**Purpose:** Blameless analysis of a production incident — what happened, why, and how to prevent recurrence. Also called post-mortem or RCA (root cause analysis).

**Key Components:**
- Incident summary (what happened, duration, impact)
- Timeline of events
- Root cause analysis (5 Whys or fishbone)
- Contributing factors
- What went well during response
- What could be improved
- Action items with owners and deadlines
- Follow-up review date

**Owner:** Incident Commander or Engineering Manager
**Audience:** Engineering, leadership
**When:** Within 48 hours of incident resolution
**Template:** `templates/06-operations/PIR.md`

**References:**
- Google SRE Book — Chapter 15: Postmortem Culture — [sre.google](https://sre.google/sre-book/postmortem-culture/)
- Etsy's blameless postmortem guide — [etsy.com/codeascraft](https://www.etsy.com/codeascraft/blameless-postmortems)
- "The Field Guide to Understanding 'Human Error'" by Sidney Dekker

---

## Phase 7: Cross-Cutting Documents

*These documents apply across multiple phases and serve the entire project lifecycle.*

---

### 7.1 KT — Knowledge Transfer / Handover Document

**Purpose:** Comprehensive handover document for transitioning a system, project, or role to another person or team.

**Key Components:**
- System overview and architecture
- Audience-specific sections (stakeholders, vendors, developers)
- Access and credentials reference
- Key processes and workflows
- Current status and pending items
- Environment and deployment details
- Troubleshooting guide
- Contact matrix

**Owner:** Outgoing team or developer
**Audience:** Incoming team, stakeholders
**When:** During team transitions, project handovers
**Template:** `templates/07-cross-cutting/KT_HANDOVER.md`

**References:**
- No formal standard — this is typically organization-specific
- GitLab's handbook transition process — [handbook.gitlab.com](https://handbook.gitlab.com)

---

### 7.2 Onboarding Guide

**Purpose:** Helps a new team member become productive as quickly as possible — accounts, tools, codebase orientation, and team norms.

**Key Components:**
- Welcome and team overview
- Account setup checklist (email, Git, cloud, tools)
- Development environment setup
- Codebase orientation (repos, structure, key files)
- Architecture overview (link to SAD/HLD)
- Team norms (meetings, communication, PR process)
- Key contacts
- First-week tasks / starter tickets
- Learning resources

**Owner:** Engineering Manager or Tech Lead
**Audience:** New hires, new team members
**When:** Maintained continuously
**Template:** `templates/07-cross-cutting/ONBOARDING.md`

**References:**
- GitLab's public handbook — [handbook.gitlab.com](https://handbook.gitlab.com)
- "An Elegant Puzzle" by Will Larson (Stripe Press) — Chapter on onboarding

---

### 7.3 Risk Register

**Purpose:** Tracks all identified project risks, their likelihood, impact, mitigation strategies, and owners.

**Key Components:**
- Risk ID and description
- Category (technical, schedule, resource, external)
- Probability (low / medium / high)
- Impact (low / medium / high / critical)
- Risk score (probability × impact)
- Mitigation strategy
- Contingency plan
- Owner
- Status (open / mitigated / closed / occurred)
- Review date

**Owner:** Project Manager
**Audience:** Project team, stakeholders
**When:** Created at project start, reviewed regularly
**Template:** `templates/07-cross-cutting/RISK_REGISTER.md`

**References:**
- PMBOK Guide — Chapter 11: Project Risk Management
- ISO 31000:2018 — Risk Management — Guidelines

---

### 7.4 RACI Matrix

**Purpose:** Clarifies roles and responsibilities for key activities and decisions — who is Responsible, Accountable, Consulted, and Informed.

**Key Components:**
- Activity / decision list (rows)
- Stakeholder / role list (columns)
- RACI assignment per cell
- R = Responsible (does the work)
- A = Accountable (final decision maker, only one per row)
- C = Consulted (provides input)
- I = Informed (kept in the loop)

**Owner:** Project Manager
**Audience:** All project stakeholders
**When:** Project initiation, updated as team changes
**Template:** `templates/07-cross-cutting/RACI.md`

**References:**
- PMBOK Guide — Resource Management
- "RACI: The Definitive Guide" — various PM resources

---

### 7.5 Meeting Minutes / Decision Log

**Purpose:** Records discussions, decisions, and action items from meetings. Creates an audit trail of how and why decisions were made.

**Key Components:**
- Meeting date, time, attendees
- Agenda items
- Discussion summary per item
- Decisions made (with rationale)
- Action items (task, owner, deadline)
- Parking lot (deferred topics)
- Next meeting date

**Owner:** Meeting organizer or designated note-taker
**Audience:** All attendees, stakeholders
**When:** After every significant meeting
**Template:** `templates/07-cross-cutting/MEETING_MINUTES.md`

---

### 7.6 Lessons Learned / Retrospective

**Purpose:** Captures what went well, what didn't, and what to improve — either at project end or after each sprint/iteration.

**Key Components:**
- Project / sprint context
- What went well (keep doing)
- What didn't go well (stop doing)
- What to try differently (start doing)
- Root causes for key issues
- Action items with owners
- Metrics comparison (planned vs. actual)

**Owner:** Scrum Master, PM, or Tech Lead
**Audience:** Project team, future project teams
**When:** After each sprint (retro) or at project close
**Template:** `templates/07-cross-cutting/LESSONS_LEARNED.md`

**References:**
- "Agile Retrospectives" by Esther Derby and Diana Larsen (Pragmatic Bookshelf)
- Spotify Retro Kit
- FunRetrospectives — [funretrospectives.com](https://www.funretrospectives.com)

---

### 7.7 Vendor Assessment / Comparison

**Purpose:** Structured evaluation of vendor options — features, pricing, pros/cons, and recommendation.

**Key Components:**
- Evaluation criteria with weights
- Vendor profiles
- Feature comparison matrix
- Pricing comparison
- Pros and cons per vendor
- Reference checks / case studies
- Security and compliance assessment
- Recommendation with rationale

**Owner:** Tech Lead or Architect
**Audience:** Decision makers, procurement
**When:** When selecting tools, platforms, or partners
**Template:** `templates/07-cross-cutting/VENDOR_ASSESSMENT.md`

---

### 7.8 Compliance / Audit Documentation

**Purpose:** Demonstrates that the system meets regulatory, security, and organizational compliance requirements.

**Key Components:**
- Applicable regulations (GDPR, HIPAA, SOC2, ISO 27001)
- Control mapping (requirement → implementation)
- Evidence collection (logs, configs, policies)
- Access control documentation
- Data handling and privacy procedures
- Audit trail and logging
- Remediation tracking
- Certification status and renewal dates

**Owner:** Compliance Officer or Security Lead
**Audience:** Auditors, legal, leadership
**When:** Before go-live, renewed periodically
**Template:** `templates/07-cross-cutting/COMPLIANCE.md`

**References:**
- SOC 2 Trust Service Criteria — [aicpa.org](https://www.aicpa.org)
- GDPR — [gdpr.eu](https://gdpr.eu)
- ISO 27001 — [iso.org](https://www.iso.org/isoiec-27001-information-security.html)
- NIST Cybersecurity Framework — [nist.gov/cyberframework](https://www.nist.gov/cyberframework)

---

### 7.9 User Manual / End-User Guide

**Purpose:** Guides end users on how to use the product — features, workflows, and troubleshooting from the user's perspective.

**Key Components:**
- Getting started / quick start
- Feature-by-feature walkthrough
- Screenshots and annotated UI guides
- Common workflows step-by-step
- FAQ
- Troubleshooting / known issues
- Glossary
- Contact support

**Owner:** Technical Writer or Product Team
**Audience:** End users
**When:** Before release, updated with each major version
**Template:** `templates/07-cross-cutting/USER_MANUAL.md`

**References:**
- "Developing Quality Technical Information" by IBM (Pearson)
- Write the Docs community — [writethedocs.org](https://www.writethedocs.org)
- Google Technical Writing courses — [developers.google.com/tech-writing](https://developers.google.com/tech-writing)

---

### 7.10 Technical Debt Log

**Purpose:** Tracks known technical debt — shortcuts, deferred refactors, outdated dependencies, and design compromises — along with their impact, interest cost, and pay-down plan.

**Key Components:**
- Debt ID and description
- Category (code quality, architecture, infrastructure, testing, documentation)
- Origin (when and why the debt was introduced)
- Impact assessment (what breaks or degrades if left unaddressed)
- Interest cost (ongoing cost of carrying this debt — slower builds, flaky tests, increased incident risk)
- Effort estimate to resolve
- Priority (pay now / pay soon / accept for now)
- Owner
- Pay-down plan (target sprint or release)
- Status (open / in progress / resolved / accepted)

**Owner:** Tech Lead or Engineering Manager
**Audience:** Engineering team, PM, leadership
**When:** Created when debt is identified, reviewed each sprint or quarterly
**Template:** `templates/07-cross-cutting/TECHNICAL_DEBT_LOG.md`

**References:**
- Martin Fowler, "Technical Debt Quadrant" — [martinfowler.com/bliki/TechnicalDebtQuadrant.html](https://martinfowler.com/bliki/TechnicalDebtQuadrant.html)
- Ward Cunningham's original debt metaphor — [wiki.c2.com/?WardExplainsDebtMetaphor](http://wiki.c2.com/?WardExplainsDebtMetaphor)
- "Managing Technical Debt" by Philippe Kruchten, Robert Nord, Ipek Ozkaya (SEI/CMU)

---

## Quick Reference: Document Cheat Sheet

**Project type legend:** M = MVP/Startup, E = Enterprise, R = Regulated/Compliance-heavy. Dots indicate the document is recommended for that project type.

| # | Document | Phase | Owner | One-Liner | M | E | R |
|---|----------|-------|-------|-----------|---|---|---|
| 1.1 | BRD | Discovery | BA | Why are we building this? | | * | * |
| 1.2 | PRD | Discovery | PM | What should the product do? | * | * | * |
| 1.3 | FRD | Discovery | BA | What exactly must the system do? | | * | * |
| 1.4 | NFR | Discovery | Architect | How well must it perform? | | * | * |
| 1.5 | Project Charter | Discovery | PM | Authorization to start | | * | * |
| 1.6 | SOW | Discovery | PM | What work will be done for payment | | * | * |
| 1.7 | RFP/RFQ/RFI | Discovery | Procurement | Solicit vendor proposals | | * | * |
| 1.8 | Stakeholder Analysis | Discovery | PM | Who cares and how much | | * | * |
| 1.9 | Feasibility Study | Discovery | BA | Can we and should we? | | * | * |
| 2.1 | HLD | Design | Architect | Big-picture architecture | * | * | * |
| 2.2 | LLD | Design | Tech Lead | Implementation-level detail | | * | * |
| 2.3 | SAD | Design | Architect | Comprehensive tech blueprint | | * | * |
| 2.4 | ERD | Design | DB Designer | Data model and relationships | * | * | * |
| 2.5 | API Spec | Design | Backend Lead | API contract | * | * | * |
| 2.6 | DFD | Design | Analyst | How data flows through the system | | * | * |
| 2.7 | Wireframes | Design | UX Designer | UI layout and interactions | * | * | * |
| 2.8 | ADR | Design+ | Architect | Why we chose X over Y (cross-cutting) | * | * | * |
| 2.9 | Threat Model | Design | Security Lead | STRIDE analysis and attack surfaces | | * | * |
| 3.1 | SDD | Development | Developer | How to implement this feature | | * | * |
| 3.2 | Coding Standards | Development | Tech Lead | How we write code here | * | * | * |
| 3.3 | README | Development | Developer | How to set up and run the project | * | * | * |
| 3.4 | DB Schema Doc | Development | Backend Dev | Database structure and migrations | * | * | * |
| 3.5 | Branching Strategy | Development | Tech Lead | How we use Git | * | * | * |
| 3.6 | Config Management | Development | DevOps | All settings across environments | | * | * |
| 4.1 | Test Plan | Testing | QA Lead | Master plan for all testing | | * | * |
| 4.2 | Test Cases | Testing | QA Engineer | Individual test scenarios | * | * | * |
| 4.3 | UAT | Testing | BA / PO | Business user validation | | * | * |
| 4.4 | Bug Report | Testing | QA Engineer | Standardized defect tracking | * | * | * |
| 4.5 | Performance Report | Testing | Perf Engineer | Load and stress test results | | * | * |
| 4.6 | Security Assessment | Testing | Security Eng | Vulnerability findings | | * | * |
| 4.7 | RTM | Testing+ | QA Lead | Requirement-to-test mapping (cross-cutting) | | * | * |
| 5.1 | Deployment Plan | Deployment | DevOps | Step-by-step deploy instructions | | * | * |
| 5.2 | Release Notes | Deployment | PM | What changed in this release | * | * | * |
| 5.3 | Rollback Plan | Deployment | DevOps | How to revert if things break | | * | * |
| 5.4 | CI/CD Pipeline Doc | Deployment | DevOps | Pipeline stages and config | * | * | * |
| 5.5 | Change Request | Deployment | PM | Formal change approval | | * | * |
| 5.6 | Go-Live Checklist | Deployment | Release Mgr | Pre-launch verification | * | * | * |
| 6.1 | SOP | Operations | Ops Lead | Routine operational procedures | | * | * |
| 6.2 | Runbook | Operations | SRE | Response to specific scenarios | | * | * |
| 6.3 | Incident Response | Operations | SRE Lead | How we handle outages | * | * | * |
| 6.4 | SLA/SLO | Operations | SRE / PM | Reliability targets and promises | | * | * |
| 6.5 | Monitoring Config | Operations | DevOps | What we watch and alert on | * | * | * |
| 6.6 | PIR / Post-Mortem | Operations | Eng Manager | Blameless incident analysis | * | * | * |
| 7.1 | KT Handover | Cross-cutting | Outgoing team | System/role transition | | * | * |
| 7.2 | Onboarding Guide | Cross-cutting | Eng Manager | New hire ramp-up | | * | * |
| 7.3 | Risk Register | Cross-cutting | PM | Tracked risks and mitigations | | * | * |
| 7.4 | RACI Matrix | Cross-cutting | PM | Who does what | | * | * |
| 7.5 | Meeting Minutes | Cross-cutting | Note-taker | Decisions and action items | | * | * |
| 7.6 | Lessons Learned | Cross-cutting | Scrum Master | What to improve next time | * | * | * |
| 7.7 | Vendor Assessment | Cross-cutting | Tech Lead | Tool/vendor comparison | * | * | * |
| 7.8 | Compliance Doc | Cross-cutting | Security Lead | Regulatory evidence | | | * |
| 7.9 | User Manual | Cross-cutting | Tech Writer | End-user guide | * | * | * |
| 7.10 | Technical Debt Log | Cross-cutting | Tech Lead | Tracked debt and pay-down plan | | * | * |

---

*Last updated: March 2026*
