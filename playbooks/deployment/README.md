# playbooks/deployment

## Purpose
Runbooks for deploying, rolling back, and validating releases.

## What Belongs Here
- Pre-deployment checklists.
- Step-by-step deployment procedures.
- Rollback and validation steps.

## What Does NOT Belong Here
- Infrastructure definitions (use `knowledge/docker/` or `knowledge/kubernetes/`).
- General CI/CD theory (use `knowledge/engineering/`).

## Lifecycle
1. Create a playbook when a new service or environment is introduced.
2. Execute and refine during each deployment.
3. Update after any deployment-related incident.

## Example Entry
- `deploy-payment-service.md`: A runbook for deploying the payment service to production with canary validation.
