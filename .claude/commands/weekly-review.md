# Weekly Review

## Purpose
Review all items in `inbox/` from the past 7 days, decide what to promote, archive, or delete, and keep the system clean.

## Input
- Implicit: all files in `inbox/<YYYY>/raw-notes/` created or modified within the last 7 days.
- The user may optionally provide a focus area (e.g., "review only security items").

## Processing
1. List files in `inbox/` created or modified within the last 7 days.
2. For each file, read its content and frontmatter.
3. Categorize each item into one of:
   - **Promote** → move to `knowledge/`, `rules/`, or `playbooks/` using the `/promote` protocol.
   - **Archive** → move to `projects/archived/` with `Status: Archived` and a note explaining why.
   - **Delete** → remove only if it is obsolete, duplicated, or noise.
   - **Keep in inbox** → leave untouched if it still needs more work or context.
4. Produce a summary report in `meta/`.

## Output
- A review report written to `meta/weekly-review-YYYY-MM-DD.md`.
- The report must contain:
  - List of files reviewed.
  - Decision per file (promote / archive / delete / keep).
  - New paths for promoted or archived items.
  - Any follow-up actions or open questions.

## Example Report Structure
```markdown
# Weekly Review — 2026-06-19

## Inbox
| File | Decision | Destination / Action |
|------|----------|----------------------|
| 2026-06-17-observed-race-in-cache.md | Promote | rules/coding/avoid-cache-race.md |
| 2026-06-18-random-shell-alias.md | Archive | projects/archived/2026-06-18-random-shell-alias.md |

## Follow-up
- Need to verify the cache race fix in staging before finalizing the rule.
```

## Output Location
- `meta/weekly-review-YYYY-MM-DD.md`
- Promoted files in their respective destinations.
- Archived files in `projects/archived/`.
