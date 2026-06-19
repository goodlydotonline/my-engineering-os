# Promote

## Purpose
Move an item from `inbox/` into a permanent destination (`knowledge/`, `rules/`, or `playbooks/`) so it becomes part of the maintained engineering system.

## Input
The user provides:
- The path to an inbox file (e.g., `inbox/2026/raw-notes/2026-06-19-token-expiry-not-checked-before-refresh.md`).
- The destination category: `knowledge`, `rules`, or `playbooks`.
- Optionally, a new title, domain, or scope override.

## Processing
1. Read the inbox file.
2. Infer or confirm the `<domain>` for the destination:
   - For `knowledge`: pick from `backend`, `ai`, `vision`, `docker`, `kubernetes`, `architecture`, `engineering` (or propose a new subdir).
   - For `rules`: pick from `coding`, `prompts`, `ai-workflow`, `project-rules`.
   - For `playbooks`: pick from `debugging`, `deployment`, `root-cause-analysis`, `project-start`.
3. Determine the final path using the rule below.
4. Rewrite the content using the appropriate template from `templates/`.
5. Set `Status: Draft` unless the user explicitly asks for `Verified`.
6. Move the file to the destination path. If a file already exists there, merge or ask rather than overwrite.
7. Update `index/topics.md` or `index/tags.md` only if a new domain or tag is introduced.

## File Movement Rules
| Destination | Path Pattern | Canonical Format |
|-------------|--------------|------------------|
| `knowledge` | `knowledge/<domain>/YYYY-MM-DD-<kebab-topic>.md` | Use `templates/research.md` or `templates/learning.md` style: what, why, when to use, examples. |
| `rules`     | `rules/<domain>/<kebab-topic>.md`              | Rule card: context, rule, rationale, exceptions, examples. |
| `playbooks` | `playbooks/<domain>/<kebab-topic>.md`           | Step-by-step procedure: goal, prerequisites, steps, verification, rollback. |

- Derive `<kebab-topic>` from the title or first meaningful sentence.
- Preserve the original capture date in `Date:` or add a `Captured:` field.

## Output
- The promoted file at its new path.
- The original inbox file is removed after a successful move.
- A short confirmation with the new path.

## Example
User says: "Promote inbox/2026/raw-notes/2026-06-19-token-expiry-not-checked-before-refresh.md to rules."

Agent:
1. Reads the inbox file.
2. Determines domain: `coding` (or `backend` if appropriate).
3. Moves to `rules/coding/check-token-expiry-before-refresh.md`.
4. Rewrites content into a rule card with frontmatter.
5. Confirms: "Promoted to `rules/coding/check-token-expiry-before-refresh.md`."
