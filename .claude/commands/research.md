# Research

## Purpose
Run a structured, reproducible research task and write the results into the knowledge base.

## Input
The user provides:
- A specific research question or topic.
- Optionally, constraints: timebox, depth (quick / deep), or preferred sources.
- Optionally, existing files or URLs to seed the research.

## Processing
1. Formulate the research question into a concise query.
2. Search the web (or provided sources) for authoritative information.
3. Synthesize findings into a structured report using `templates/research.md`:
   - **问题** — restated clearly.
   - **调研** — candidate comparisons, metrics, references.
   - **结论** — direct answer or recommendation.
   - **风险** — technical and business risks plus mitigations.
   - **下一步** — action checklist.
4. Determine the best `knowledge/<domain>/` destination.
5. Write the report to the output path using the naming rule below.
6. If the research reveals actionable rules or playbooks, trigger `/promote` to move them to the appropriate destination.

## File-Naming Rule
```
knowledge/<domain>/YYYY-MM-DD-<kebab-topic>.md
```
- Use today's date.
- Derive `<domain>` from the subject (e.g., `backend`, `ai`, `docker`, `architecture`).
- Derive `<kebab-topic>` from the research question, lowercased, with punctuation stripped and spaces replaced by hyphens.
- If the file already exists, append `-N` before `.md`.

## Output Format
Use the standard frontmatter from `templates/research.md`:
```yaml
---
Title: "<topic>"
Date: YYYY-MM-DD
Tags: ["#research", "#<domain>"]
Status: Draft
Source: "<primary source>"
Related: []
---
```

## Example
User says: "Research: what are the current best practices for zero-downtime PostgreSQL migrations?"

Agent writes: `knowledge/backend/2026-06-19-zero-downtime-postgresql-migrations.md`

## Output Location
- `knowledge/<domain>/YYYY-MM-DD-<kebab-topic>.md`
- Any promoted rules or playbooks derived from the research.
