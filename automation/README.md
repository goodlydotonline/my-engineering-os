# automation

## Purpose
Scripts, workflows, and configurations for automating repetitive tasks and maintaining the engineering OS.

## What Belongs Here
- Shell scripts, Python scripts, and GitHub Actions workflows.
- Automation for linting, formatting, and indexing.
- Scheduled maintenance tasks.

## What Does NOT Belong Here
- Project-specific deployment scripts (use `playbooks/deployment/` or `projects/`).
- One-off, ad-hoc scripts.

## Lifecycle
1. Identify a repetitive task.
2. Automate and document the script or workflow.
3. Monitor and update as the environment changes.

## Example Entry
- `sync-index.py`: A script that regenerates the knowledge index based on file tags.
