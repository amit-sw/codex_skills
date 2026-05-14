# Deep Agent LLM Reference

Use this reference when a feature talks to an LLM.

## Default Rule

Use a Deep Agent style framework when the LLM must plan, inspect files/data, use tools, call APIs, branch on intermediate results, retry, critique, or execute multi-step work.

Use a direct LLM call only when the task is narrow and isolated, such as:

- Classify one bounded text input.
- Extract fields into a strict schema from one document.
- Rewrite a short string.
- Score a record using a fixed rubric with no tool use.

When unsure, choose Deep Agent for product workflows and direct calls for leaf transformations.

## Required Design

For each LLM workflow, define:

- User-visible purpose and failure mode.
- Agent goal, allowed tools, disallowed tools, and stopping conditions.
- Input and output schemas with Zod or equivalent structured validation.
- State model: what is persisted in D1, what large context/artifacts are stored in R2, and what is transient.
- Trace model: every Agentic workflow must stream its trace to R2 during execution, with D1 pointers for lookup and replay.
- Audit model: prompt/template version, model, tool calls, user/admin actor, effective user, request ID, and output reference.
- Observability: trace/run ID, token usage where available, latency, tool-call counts, error class, retry count.
- Evaluation: deterministic fixtures, golden outputs where appropriate, adversarial cases, and regression checks.

## Execution Rules

- Keep prompts/templates in source control unless product users edit them. If editable, store compact prompt metadata in D1 and full versions in R2 when large.
- Prefer structured outputs. Do not parse free-form prose when a schema can define the result.
- Persist enough inputs and artifacts to replay or audit important decisions.
- For Agentic workflows, stream trace events to R2 as they happen. Do not rely on a final write after completion because failed runs must still be replayable.
- Trace events should include run ID, step sequence, surfaced planning/decision state, tool calls, tool results or redacted summaries, model/provider metadata, input/output schema versions, retry/error events, and object references for large artifacts.
- Store only lookup metadata in D1: run ID, workflow name/version, actor/effective user, request/job ID, status, failure summary, trace R2 key, created/updated timestamps, and retention class.
- Provide internal replay tooling or a runbook that can load a trace by run ID immediately after a support report and reconstruct the agent's execution path without production database spelunking.
- Treat vendor memory/vector stores as rebuildable. Keep source material in R2 and state in D1.
- Put long-running agent work behind a queue. The synchronous request creates a durable job and returns status.
- Surface progress via polling or SSE using compact status from D1.

## Safety and Operations

- Define tool permissions by role and workflow. Agents must not gain admin powers implicitly.
- For admin-initiated LLM actions affecting a user, audit both identities.
- Redact secrets before sending context to the LLM and before writing traces to R2.
- Log model/provider errors as structured summaries, not raw secret-bearing payloads.
- Restrict trace replay access to internal support/engineering roles and audit trace access for sensitive workflows.
- Add a kill switch or feature flag for high-risk LLM workflows.
