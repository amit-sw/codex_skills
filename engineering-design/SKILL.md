---
name: engineering-design
description: Design or redesign small-team, mission-critical Cloudflare-first services that target 99.9% availability, automatic GitHub-to-production deploys, strong tests, observability, runbooks, admin impersonation audit trails, R2/D1 storage discipline, and Deep Agent based LLM workflows.
---

# Mission-Critical Cloudflare Engineering Design

Use this skill before building or substantially changing a production service where reliability, observability, admin operations, auditability, or LLM-driven workflows matter. It is optimized for a small team that wants simple architecture, high automation, automatic production deploys from GitHub check-ins, and fast Cloudflare rollback.

## Core Defaults

- Target **99.9% availability** unless the user sets a different SLO. Treat this as about 43 minutes of monthly error budget.
- Prefer the Cloudflare stack: one Worker serving React + Vite assets, Hono API routes, and queue consumers; D1 for small transactional state; R2 for large content; Queues for async work; Cloudflare Access for auth.
- Use TypeScript end-to-end with Zod schemas at HTTP, queue, and LLM boundaries.
- Use **Deep Agent style orchestration** for most LLM workflows where the model needs to plan, use tools, inspect state, or execute multi-step work. Use a direct single LLM call only for narrow, isolated transformations.
- Stream every Agentic workflow trace to R2 as the workflow runs. Store D1 pointers and metadata so internal support can replay the agent's execution path immediately when a failure is reported.
- Keep the client thin. The Worker derives identity from Cloudflare Access and performs all authorization server-side.
- Deploy automatically to production on successful GitHub checks. Favor strong pre-merge checks, canary/synthetic verification where practical, and documented Cloudflare rollback over heavyweight release ceremonies.
- Maintain `spec.md` or equivalent living design notes for major changes.
- Maintain `ChangeLog.md` in this skill folder with all major changes to the skill.
- Before making changes with significant architecture, future maintainability, or observability impact, explain the ramifications and get explicit user confirmation that the tradeoff is acceptable.

## Quick Workflow

1. **Classify the change**: new service, major redesign, risky feature, admin capability, LLM workflow, storage migration, or production operations change.
2. **Pause for material tradeoffs** before changing architecture, long-term maintainability, or observability. Tell the user the ramifications, alternatives, and operational cost, then wait for explicit confirmation that the tradeoff is acceptable.
3. **Define SLOs and critical flows** before architecture. Identify SLIs, error budget, user-visible failure modes, and non-goals.
4. **Choose the Cloudflare architecture** and document any deviations. Read `references/cloudflare-runtime.md` when storage, queues, auth, or deployment topology matter.
5. **Decompose components** into routes, services, repositories, adapters, shared schemas, client views, queues, and operational scripts.
6. **Design the LLM execution model**. Read `references/deep-agent-llm.md` when the feature uses an LLM for anything beyond a simple bounded call.
7. **Define tests and release gates**. Read `references/testing-and-ci.md` for required automation.
8. **Define observability and operations**. Read `references/slo-observability-operations.md` for telemetry, alerts, dashboards, and runbooks.
9. **Define admin and audit behavior**. Read `references/admin-audit-impersonation.md` for impersonation and privileged actions.
10. **Complete production readiness**. Read `references/production-readiness.md` before implementation or launch.

## Design Rules

- D1 stores small transactional state only: IDs, owners, statuses, timestamps, compact metadata, counters, audit events, and R2 object pointers.
- R2 stores large content: uploads, PDFs, images, exports, transcripts, large prompts/responses, generated files, and replayable raw artifacts.
- Queue messages carry IDs and small routing metadata only. Consumers rehydrate state from D1 and bytes from R2.
- Agentic workflow traces must stream to R2, not only flush at completion. D1 stores run IDs, status, timestamps, actor/effective user IDs, workflow version, failure summary, and R2 trace object keys.
- Repositories own D1 SQL. Adapters own external systems and Cloudflare bindings. Services compose workflows. Routes validate and delegate.
- Every user-owned query must scope by verified owner identity. The client must never send trusted `user_id`.
- Any privileged operation must produce an audit record. Admin impersonation must record both identities: the admin actor and the effective user.
- Logs must be structured and must never include secrets, access tokens, raw private keys, or unnecessary PII.
- Missing dependencies, credentials, bindings, or required configuration must fail fast with clear diagnostics.

## Required Deliverables

- `spec.md` or design note covering goals, non-goals, SLOs, critical flows, architecture, data model, storage split, LLM model, failure modes, and rollout.
- Component matrix with each route/service/repository/adapter/queue/client area, owner identity rules, dependencies, and tests.
- D1 schema plan and R2 object key plan, including retention and lifecycle expectations.
- Agentic trace streaming and replay plan covering R2 bucket/key structure, D1 pointers, retention, redaction, replay tooling, and internal support access.
- CI plan with hands-off tests and GitHub checks required before automatic production deploy.
- Observability plan with structured logs, request IDs, metrics, traces where available, dashboards, alerts, and synthetic checks.
- Runbooks/checklists for deploy, rollback, incident response, failed queue/DLQ handling, migration recovery, and admin impersonation review.
- Confirmed tradeoff notes for any significant architecture, future maintainability, or observability impact, including the user confirmation.
- Production readiness checklist with explicit go/no-go status.

## Skill Maintenance

- Record every major change to this skill in `ChangeLog.md` with the date, summary, and affected files or guidance areas.
- Keep changelog entries concise and useful for future maintainers; do not duplicate full reference text.

## References

- `references/cloudflare-runtime.md` - Cloudflare stack, storage split, queues, Access auth, and deployment defaults.
- `references/deep-agent-llm.md` - when and how to use Deep Agent style LLM planning/execution.
- `references/testing-and-ci.md` - small-team automated test and GitHub deployment gates.
- `references/slo-observability-operations.md` - SLOs, telemetry, alerts, runbooks, and incident process.
- `references/admin-audit-impersonation.md` - privileged admin workflows and audit trail requirements.
- `references/production-readiness.md` - final readiness checklist before implementation or production launch.
