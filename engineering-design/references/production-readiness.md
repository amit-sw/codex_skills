# Production Readiness Reference

Use this before implementation starts for high-risk changes and before production launch for new services.

## Readiness Checklist

Mark every item as `ready`, `not applicable`, or `blocked`. Do not delete items.

### Product and Scope

- Critical user flows are named.
- Non-goals and exclusions are explicit.
- Availability target and error budget are defined.
- Business owner and technical owner are named.

### Architecture

- Cloudflare default stack is used or deviations are justified.
- Significant architecture or future-maintainability tradeoffs were explained to the user and explicitly confirmed before implementation.
- One Worker deployable is preserved unless there is a clear operational requirement.
- Routes/services/repositories/adapters boundaries are documented.
- D1/R2/Queues split is documented.
- Large content is stored in R2, not D1.
- Queue jobs are idempotent and have DLQ handling.

### Auth, Admin, and Audit

- Cloudflare Access auth is verified in Worker middleware.
- Authorization is server-side and repository-scoped.
- Admin roles and privileged actions are defined.
- Impersonation records both admin actor and effective user.
- Audit logs cover privileged actions and user-impacting changes.
- Secret masking is implemented and tested.

### LLM Workflows

- Deep Agent is used for broad planning/execution workflows.
- Direct LLM calls are limited to narrow leaf transformations.
- Prompts, tools, schemas, and stop conditions are documented.
- Agentic workflow traces stream to R2 during execution and preserve partial traces for failed runs.
- LLM runs are traceable and replayable enough for internal support to reconstruct the agent's execution path immediately from a run ID, request ID, or job ID.
- Large LLM artifacts are stored in R2 with D1 pointers.

### Tests and CI

- Local hands-off test command exists.
- Cloudflare Workers' GitHub integration runs typecheck, lint, tests, build, and deployment validation.
- No GitHub Actions workflows are used for CI/CD unless the user explicitly approved a documented exception.
- Auth, audit, queue, migration, R2, and LLM failure paths are tested.
- Critical flows have smoke or e2e tests.
- Full Production Tests are defined for critical flows, safe to run in production, and require only a logged-in authorized user or dedicated production test identity.
- Admin users can run Full Production Tests with one click and see prior run results.
- Cloudflare Workers GitHub validation passing triggers production deploy.

### Observability and Operations

- Structured logs include request/job IDs and outcomes.
- Significant observability tradeoffs were explained to the user and explicitly confirmed before implementation.
- Dashboards cover critical flows, errors, latency, queues, DLQ, R2, D1, and LLM calls.
- Alerts exist for SLO burn, production deploy failure, synthetic failure, Full Production Test failure, queue/DLQ problems, and audit failures.
- Runbooks exist for incident response, rollback, DLQ drain, migration failure, Access failure, R2 object issue, and LLM degradation.
- Runbooks exist for internal Agentic workflow failure reports, including trace lookup and replay from R2.
- Post-incident review process includes updating tests and runbooks.

### Deployment and Rollback

- Production deploy is automatic through Cloudflare Workers' GitHub integration.
- Post-deploy synthetic validation exists.
- Full Production Tests run after deploy when practical or are available for immediate admin-triggered validation.
- Cloudflare rollback command/UI path is documented.
- Owner is named for responding to failed deploys.
- Migration risk and recovery plan are documented.

## Go/No-Go Rule

For a mission-critical service, do not proceed when any of these are blocked:

- Critical flow SLO definition.
- User-confirmed acceptance of any significant architecture, future-maintainability, or observability tradeoff.
- Auth/authorization model.
- Audit trail for privileged actions.
- D1/R2 storage split.
- Cloudflare Workers GitHub integration gate for automatic deploy.
- Full Production Tests definition, admin run path, stored results, and failure alerting.
- Rollback runbook.
- Observability for user-visible failures.
- R2 trace streaming and replay for production Agentic workflows.

Small-team speed comes from simple defaults and automation, not skipping the reliability basics.
