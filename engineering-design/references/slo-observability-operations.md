# SLO, Observability, and Operations Reference

Use this reference when defining reliability goals, telemetry, alerts, dashboards, and runbooks.

## SLO Design

Start with user-visible critical flows. Examples:

- Authenticated app load.
- Create/update/delete core entity.
- Upload content to R2 and create D1 metadata.
- Enqueue and complete async processing.
- Admin impersonation action.
- LLM job completion.

For each flow define:

- SLI: availability, latency, correctness, freshness, or durability.
- SLO target, normally 99.9% for availability.
- Measurement window, normally 28 or 30 days.
- Error budget and what consumes it.
- Explicit exclusions.
- Alert threshold and owner.

For 99.9%, a 30-day window has about 43 minutes of total error budget.

## Telemetry Contract

Every request/job should have:

- `request_id` or `trace_id`.
- `user_id` or service identity when available.
- `admin_actor_id` and `effective_user_id` when impersonating.
- Route/job name.
- Status/outcome.
- Duration.
- Error class and compact message.
- R2 key references, never raw large content.
- Model/provider metadata for LLM calls where relevant.
- Agentic workflow trace R2 key and replay status where relevant.

Use structured logs. Do not log secrets, tokens, private keys, raw credential objects, or unnecessary PII.

For Agentic workflows, structured logs are not enough. Stream the execution trace to R2 as the workflow runs, and include the trace key in D1/job metadata and operational logs. Failed runs must preserve partial traces so internal support can replay the agent's execution path immediately.

## Cloudflare Observability

Use Workers Observability for invocation logs, custom logs, errors, CPU time, wall time, and execution duration. Export OpenTelemetry-compatible traces/logs if the service needs longer retention or cross-system correlation.

Plan dashboards for:

- Request volume, error rate, latency.
- Critical flow success rate.
- Queue depth, retries, DLQ count, job age.
- D1 errors and slow queries where visible.
- R2 failures.
- LLM latency, error rate, token usage, and tool failures.
- Agentic trace write failures, replay lookup failures, and trace retention/lifecycle health.
- Admin impersonation count and privileged action count.

## Alerting

Alert on symptoms users feel, not only internal causes.

Required alerts:

- Critical flow availability burn.
- Sustained 5xx or auth failure spike.
- Queue backlog or oldest job age above threshold.
- Any DLQ messages in production.
- Post-deploy synthetic failure.
- Audit logging failure for privileged operations.
- Agentic trace streaming failure for production LLM workflows.
- LLM provider outage affecting critical flows.

Prefer burn-rate style alerts for SLO-backed flows where the signal supports it.

## Runbooks

Required runbooks/checklists:

- First 10 minutes of an incident.
- Cloudflare rollback.
- Failed automatic deploy.
- D1 migration failure or data issue.
- Queue backlog and DLQ drain.
- R2 object missing/corrupt.
- Access/auth failure.
- Admin impersonation review.
- LLM workflow degradation or provider outage.
- Internal failure report: locate agent run by user/job/request ID, load R2 trace, replay execution path, identify failing step, and preserve evidence for follow-up.

Each runbook must include detection, impact, owner, immediate mitigation, validation, and follow-up.
