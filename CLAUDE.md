# CLAUDE.md — Agent Context for Engineering OS

> This file provides persistent context for Claude Code (and other agents) when working inside this repository.

---

## Project Identity

- **Name:** Personal Engineering Operating System
- **Type:** AI-first, Markdown-first, long-term knowledge system
- **Goal:** Convert raw engineering inputs into reusable, verified knowledge and rules.
- **Non-goal:** This is not a generic notes dump or a public blog.

---

## Directory Map (Canonical)

| Directory | Purpose |
| :--- | :--- |
| `inbox/` | Raw, unprocessed inputs only. Status: `Inbox`. |
| `knowledge/` | Verified, reusable technical knowledge. Status: `Verified` or `Draft`. |
| `rules/` | Prescriptive engineering rules for agents and humans. Status: `Verified` or `Draft`. |
| `playbooks/` | Step-by-step runbooks for debugging, deployment, RCA, and project start. |
| `projects/` | Active and archived project-specific work. |
| `projects/archived/` | Completed or deprecated projects. |
| `templates/` | Blank templates for bug reports, postmortems, research, and learning. |
| `index/` | Navigation: `topics.md`, `tags.md`, `roadmap.md`. |
| `automation/` | Future scripts, GitHub Actions, hooks, and maintenance bots. |
| `meta/` | Changelog, proposals, and health reports about the OS itself. |
| `.claude/commands/` | Executable agent protocols: `capture`, `promote`, `weekly-review`, `research`. |

---

## Agent Rules of Engagement

1. **Read before writing.** Check `knowledge/` and `rules/` in the relevant domain before creating new content.
2. **Draft in `inbox/` first.** Do not write directly to `knowledge/` or `rules/` unless explicitly told to promote.
3. **Use templates.** Start every new Markdown file with the standard frontmatter from `templates/`.
4. **Frontmatter is mandatory.** For any file in `knowledge/`, `rules/`, `playbooks/`, `projects/`:
   ```yaml
   ---
   Title: "..."
   Date: YYYY-MM-DD
   Tags: ["#tag"]
   Status: "Inbox | Draft | Verified | Archived"
   Source: "..."
   Related: ["path/to/file.md"]
   ---
   ```
5. **Link aggressively.** Cross-reference related files.
6. **Flag conflicts.** If a request contradicts an existing rule, stop and ask.
7. **Archive, do not delete.** When deprecating, move to `projects/archived/` or set `Status: Archived`.
8. **Prefer Chinese headings in templates.** The templates use Chinese section names (`问题`, `现象`, etc.); preserve them unless told otherwise.

---

## Available Slash Commands

When the user invokes one of these, read the corresponding file and execute exactly:

- `/capture` → `.claude/commands/capture.md`
- `/promote` → `.claude/commands/promote.md`
- `/weekly-review` → `.claude/commands/weekly-review.md`
- `/research` → `.claude/commands/research.md`

---

## Review Cadence

- **Daily:** Process `inbox/`.
- **Weekly:** Verify drafts, resolve rule conflicts, archive stale items.
- **Monthly:** Audit `Verified` content, update this file and agent prompts.

---

## Tag Conventions

Use the canonical tag set in `index/tags.md`. Add new tags only after updating `index/tags.md` and the README tag table.

---

## Prohibited Actions

- Do not create top-level directories not listed above without human approval.
- Do not store secrets, credentials, or company-internal IPs here.
- Do not commit generated artifacts without a clear source note.
