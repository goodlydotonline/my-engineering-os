# topics.md

## Purpose
Domain index for the Engineering OS. Maps each engineering domain to the directories that actually exist in this repository and provides the primary navigation entry point.

---

## Backend
Server-side systems, APIs, databases, and service architecture.
- Directories: [`knowledge/backend/`](../knowledge/backend/)
- Tags: `#backend`, `#infra`

## AI
Machine learning, LLMs, prompt engineering, and agentic systems.
- Directories: [`knowledge/ai/`](../knowledge/ai/)
- Tags: `#ai`, `#llm`

## Vision
Computer vision, image processing, and visual perception.
- Directories: [`knowledge/vision/`](../knowledge/vision/), [`knowledge/vision/ar-measurement/`](../knowledge/vision/ar-measurement/)
- Tags: `#vision`, `#cv`, `#ar`, `#measurement`

## Docker
Containerization, image builds, and runtime configuration.
- Directories: [`knowledge/docker/`](../knowledge/docker/)
- Tags: `#docker`, `#container`, `#infra`

## Kubernetes
Orchestration, cluster management, and cloud-native deployments.
- Directories: [`knowledge/kubernetes/`](../knowledge/kubernetes/)
- Tags: `#k8s`, `#orchestration`, `#infra`

## Architecture
System design, patterns, and structural decisions.
- Directories: [`knowledge/architecture/`](../knowledge/architecture/), [`projects/`](../projects/)
- Tags: `#architecture`, `#design`, `#patterns`

## Engineering
Core software engineering practices, workflows, and tooling.
- Directories: [`knowledge/engineering/`](../knowledge/engineering/), [`playbooks/`](../playbooks/)
- Tags: `#engineering`, `#tooling`, `#workflow`

## Debugging
Diagnostic techniques, tools, and systematic troubleshooting.
- Directories: [`playbooks/debugging/`](../playbooks/debugging/)
- Tags: `#debug`, `#troubleshoot`, `#observability`

## Deployment
Release pipelines, environments, and delivery workflows.
- Directories: [`playbooks/deployment/`](../playbooks/deployment/)
- Tags: `#deploy`, `#cicd`, `#release`

## Root Cause Analysis
Post-mortems, incident analysis, and systematic failure investigation.
- Directories: [`playbooks/root-cause-analysis/`](../playbooks/root-cause-analysis/)
- Tags: `#postmortem`, `#rca`, `#incident`

## Project Start
Bootstrapping, scoping, and kickoff procedures.
- Directories: [`playbooks/project-start/`](../playbooks/project-start/), [`templates/`](../templates/)
- Tags: `#project-start`, `#bootstrap`, `#scoping`

## Coding
Implementation patterns, style guides, and language-specific rules.
- Directories: [`rules/coding/`](../rules/coding/)
- Tags: `#coding`, `#language`, `#style`

## Prompts
LLM prompt libraries, reusable templates, and prompt-engineering conventions.
- Directories: [`rules/prompts/`](../rules/prompts/)
- Tags: `#prompt`, `#llm`, `#template`

## AI Workflow
Agentic workflows, AI-assisted development, and automation patterns.
- Directories: [`rules/ai-workflow/`](../rules/ai-workflow/), [`automation/`](../automation/)
- Tags: `#ai-workflow`, `#agent`, `#automation`

## Project Rules
Project-specific conventions, constraints, and operational rules.
- Directories: [`rules/project-rules/`](../rules/project-rules/), [`meta/`](../meta/)
- Tags: `#rule`, `#convention`, `#constraint`

---

## How to Extend
1. When a domain accumulates 3+ entries, consider creating a dedicated subdirectory under `knowledge/` or `playbooks/`.
2. Update this file with the new directory link and relevant tags.
3. Add new tags to `index/tags.md` before using them in content.
