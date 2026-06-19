# Capture

## Purpose
Capture a raw insight, artifact, error log, or note into the inbox so nothing is lost before it is processed.

## Input
The user provides one of the following:
- A free-form text note or insight.
- A pasted code snippet, error log, or command output.
- A file path to copy into the inbox.
- A URL (optional) to record as source context.

## Processing
1. Read the provided content.
2. Generate a filename using the rule below.
3. Write the content to `inbox/<YYYY>/raw-notes/<filename>` with the frontmatter below.
4. If a source URL or original file path was provided, record it in `Source:`.

## File-Naming Rule
```
inbox/<YYYY>/raw-notes/YYYY-MM-DD-<kebab-topic>.md
```
- Use today's date.
- Derive `<kebab-topic>` from the first 4-6 meaningful words of the content, lowercased, with punctuation stripped and spaces replaced by hyphens.
- If the file already exists, append `-N` (e.g., `-1`, `-2`) before `.md`.

## Output Format
Each inbox file must start with YAML frontmatter and have `Status: Inbox`:
```yaml
---
Title: "<concise title>"
Date: YYYY-MM-DD
Tags: ["#draft"]
Status: Inbox
Source: "<url | file-path | user-paste>"
Related: []
---

<raw content>
```

## Example
User says: "Capture: while debugging auth we noticed the token expiry is not checked before the refresh call."

Agent writes: `inbox/2026/raw-notes/2026-06-19-token-expiry-not-checked-before-refresh.md`

```yaml
---
Title: "Token expiry not checked before refresh call"
Date: 2026-06-19
Tags: ["#draft"]
Status: Inbox
Source: "user-paste"
Related: []
---

While debugging auth we noticed the token expiry is not checked before the refresh call.
```

## Output Location
- `inbox/<YYYY>/raw-notes/YYYY-MM-DD-<kebab-topic>.md`
