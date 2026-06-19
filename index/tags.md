# tags.md

## Purpose
Unified tag index for the Engineering OS. Defines the canonical tag set, meanings, and example usage to ensure consistent labeling across documents.

**Rule:** Tags live in the YAML frontmatter of every Markdown file (`Tags: ["#tag"]`). Inline `#hashtags` are optional for search but frontmatter is the source of truth.

---

## Tag Set

### Domain Tags

#### #bug
A documented bug, its reproduction, or its fix.
- Example: `knowledge/backend/2024-06-19-memory-leak.md` with `Tags: ["#bug"]`

#### #infra
Infrastructure, DevOps, and operational concerns.
- Example: `knowledge/docker/multi-stage-builds.md` with `Tags: ["#infra"]`

#### #docker
Docker-specific content including images, containers, and Compose.
- Example: `knowledge/docker/debug-container.md` with `Tags: ["#docker"]`

#### #container
Container runtime and image concerns.
- Example: `knowledge/docker/image-layer-caching.md` with `Tags: ["#container", "#docker"]`

#### #k8s
Kubernetes, orchestration, and cluster management.
- Example: `knowledge/kubernetes/rolling-update.md` with `Tags: ["#k8s"]`

#### #orchestration
Scheduling, service mesh, and cluster orchestration beyond Kubernetes.
- Example: `knowledge/kubernetes/service-mesh-comparison.md` with `Tags: ["#orchestration", "#k8s"]`

#### #architecture
System design, structural decisions, and patterns.
- Example: `knowledge/architecture/event-driven-design.md` with `Tags: ["#architecture"]`

#### #design
Design trade-offs and heuristics.
- Example: `knowledge/architecture/api-versioning.md` with `Tags: ["#design", "#architecture"]`

#### #patterns
Reusable architectural or implementation patterns.
- Example: `knowledge/architecture/cqrs-pattern.md` with `Tags: ["#patterns", "#architecture"]`

#### #ai
Artificial intelligence, machine learning, and LLMs.
- Example: `knowledge/ai/prompt-engineering-guide.md` with `Tags: ["#ai"]`

#### #llm
Large language model specifics, APIs, and tuning.
- Example: `rules/prompts/chain-of-thought.md` with `Tags: ["#llm", "#ai"]`

#### #vision
Computer vision, image processing, and visual perception.
- Example: `knowledge/vision/camera-calibration.md` with `Tags: ["#vision"]`

#### #backend
Server-side APIs, services, and databases.
- Example: `knowledge/backend/rest-api-versioning.md` with `Tags: ["#backend"]`

#### #frontend
Browser, React, CSS, and client-side concerns.
- Example: `knowledge/engineering/frontend-performance.md` with `Tags: ["#frontend"]`

#### #database
SQL, NoSQL, schema design, and query optimization.
- Example: `knowledge/backend/postgresql-indexing.md` with `Tags: ["#database", "#backend"]`

#### #security
Auth, encryption, vulnerabilities, and secure defaults.
- Example: `rules/coding/validate-jwt-signature.md` with `Tags: ["#security"]`

#### #performance
Benchmarks, optimization, and profiling.
- Example: `knowledge/backend/redis-pipelining.md` with `Tags: ["#performance", "#backend"]`

#### #testing
Unit tests, integration tests, TDD, and quality practices.
- Example: `rules/coding/write-unit-tests-for-edge-cases.md` with `Tags: ["#testing"]`

### Workflow Tags

#### #debug
Debugging techniques, tools, and diagnostic workflows.
- Example: `playbooks/debugging/debug-docker-network.md` with `Tags: ["#debug"]`

#### #troubleshoot
Operational troubleshooting and incident handling.
- Example: `playbooks/debugging/intermittent-502.md` with `Tags: ["#troubleshoot", "#debug"]`

#### #observability
Metrics, logs, traces, and monitoring.
- Example: `knowledge/engineering/otel-tracing.md` with `Tags: ["#observability"]`

#### #deploy
Release pipelines and delivery workflows.
- Example: `playbooks/deployment/zero-downtime-deploy.md` with `Tags: ["#deploy"]`

#### #cicd
Continuous integration and delivery.
- Example: `playbooks/deployment/github-actions-patterns.md` with `Tags: ["#cicd", "#deploy"]`

#### #release
Release management, versioning, and rollbacks.
- Example: `playbooks/deployment/rollback-checklist.md` with `Tags: ["#release", "#deploy"]`

#### #postmortem
Incident post-mortems and root cause analysis reports.
- Example: `playbooks/root-cause-analysis/2024-05-outage.md` with `Tags: ["#postmortem"]`

#### #rca
Root cause analysis methodology.
- Example: `playbooks/root-cause-analysis/five-whys-template.md` with `Tags: ["#rca", "#postmortem"]`

#### #incident
Live incident response and communication.
- Example: `playbooks/root-cause-analysis/incident-commander-checklist.md` with `Tags: ["#incident"]`

#### #project-start
Bootstrapping, scoping, and kickoff procedures.
- Example: `playbooks/project-start/new-service-checklist.md` with `Tags: ["#project-start"]`

#### #bootstrap
Project or repo bootstrap tasks.
- Example: `playbooks/project-start/python-service-template.md` with `Tags: ["#bootstrap", "#project-start"]`

#### #scoping
Scope definition and RFC drafting.
- Example: `playbooks/project-start/rfc-template.md` with `Tags: ["#scoping", "#project-start"]`

#### #engineering
Core software engineering practices and workflows.
- Example: `knowledge/engineering/code-review-checklist.md` with `Tags: ["#engineering"]`

#### #tooling
Editors, linters, CLI tools, and workflow automation.
- Example: `knowledge/engineering/vim-tips.md` with `Tags: ["#tooling", "#engineering"]`

#### #workflow
Process and workflow design.
- Example: `rules/ai-workflow/prompt-before-refactor.md` with `Tags: ["#workflow", "#ai-workflow"]`

#### #ai-workflow
Agentic workflows and AI-assisted development.
- Example: `rules/ai-workflow/weekly-review-protocol.md` with `Tags: ["#ai-workflow"]`

#### #agent
Agent configuration, prompts, and behavior.
- Example: `rules/ai-workflow/claude-context-rules.md` with `Tags: ["#agent", "#ai-workflow"]`

#### #automation
Scripts, hooks, and scheduled tasks.
- Example: `automation/sync-index.py` with `Tags: ["#automation"]`

### Content-Type Tags

#### #rule
Project rules, conventions, and constraints.
- Example: `rules/coding/python-style.md` with `Tags: ["#rule", "#coding"]`

#### #coding
Language-specific implementation knowledge and rules.
- Example: `rules/coding/python-type-hints.md` with `Tags: ["#coding", "#python"]`

#### #language
Programming language specifics.
- Example: `knowledge/backend/rust-ownership.md` with `Tags: ["#language", "#backend"]`

#### #python
Python-specific content.
- Example: `rules/coding/use-ruff.md` with `Tags: ["#python", "#coding"]`

#### #typescript
TypeScript/JavaScript-specific content.
- Example: `rules/coding/ts-strict-mode.md` with `Tags: ["#typescript", "#coding"]`

#### #style
Style guides and formatting rules.
- Example: `rules/coding/formatting.md` with `Tags: ["#style", "#coding"]`

#### #prompt
Prompt libraries and prompt-engineering conventions.
- Example: `rules/prompts/code-review-prompt.md` with `Tags: ["#prompt", "#llm"]`

#### #template
Templates and scaffolding.
- Example: `templates/bug-report.md` with `Tags: ["#template"]`

#### #convention
Team or personal conventions.
- Example: `rules/project-rules/naming-conventions.md` with `Tags: ["#convention", "#rule"]`

#### #constraint
Hard constraints and non-functional requirements.
- Example: `rules/project-rules/no-pii-in-logs.md` with `Tags: ["#constraint", "#rule", "#security"]`

#### #learning
Learning notes, tutorials, and skill acquisition.
- Example: `knowledge/backend/rust-ownership.md` with `Tags: ["#learning", "#language"]`

#### #research
Structured research outputs.
- Example: `knowledge/backend/2026-06-19-postgresql-migrations.md` with `Tags: ["#research", "#database"]`

#### #career
Growth, retrospectives, and soft skills.
- Example: `meta/roadmap.md` with `Tags: ["#career"]`

### Status Tags

#### #draft
Work in progress, not yet reviewed or finalized.
- Example: `knowledge/backend/rust-ownership.md` with `Tags: ["#draft"]` while `Status: Draft`

#### #verified
Reviewed and validated content.
- Example: `knowledge/architecture/cqrs-pattern.md` with `Tags: ["#verified"]` while `Status: Verified`

#### #archived
Deprecated or superseded content kept for reference.
- Example: `projects/archived/2025-legacy-migration.md` with `Tags: ["#archived"]` while `Status: Archived`

#### #meta
About the Engineering OS itself.
- Example: `README.md` with `Tags: ["#meta"]`

---

## Usage Rules
1. Use lowercase, hyphenated tags only.
2. Prefer tags from this canonical set; add new tags only after updating this file.
3. Put tags in the YAML frontmatter (`Tags: ["#tag1", "#tag2"]`).
4. Combine domain + content-type + status tags as needed (e.g., `Tags: ["#docker", "#rule", "#verified"]`).
5. Propose new tags in `inbox/tag-proposals.md` and formalize them during weekly review.
