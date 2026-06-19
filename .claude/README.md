# .claude

## Purpose
Claude Code-specific configuration and custom commands for this repository.

## What Belongs Here
- Claude settings and custom slash commands.
- Project-specific Claude Code configurations.

## What Does NOT Belong Here
- General project documentation (use `projects/` or `knowledge/`).
- Secrets or credentials.

## Lifecycle
1. Add configurations as Claude Code features are adopted.
2. Review and update with tool and model changes.
3. Keep synchronized with team practices.

## Example Entry
- `commands/`: Custom slash commands for this project.

## Commands

This directory contains Claude command protocol files. Each file is a standalone instruction set that an agent can execute directly.

### How to use
Run a command by referencing its name (without the `.md` extension). The agent reads the file and follows the protocol.

### Available commands

| Command | Purpose |
|---------|---------|
| `capture` | Capture a raw insight or artifact into the inbox. |
| `promote` | Promote an inbox item into `knowledge/`, `rules/`, or `playbooks/`. |
| `weekly-review` | Review inbox and drafts from the past week; promote, archive, or delete. |
| `research` | Run a structured research task and write results into the knowledge base. |

### Adding new commands
1. Create a new `<command-name>.md` file in this directory.
2. Include sections: Purpose, Prerequisites, Input, Processing, Output, and Example.
3. Keep the command directly executable by an agent without extra explanation.
