# Cloudflare Runtime Reference

Use this reference when designing storage, async jobs, auth, or deployment topology.

## Standard Shape

- One deployable Cloudflare Worker serves static SPA assets, JSON API routes, SSE endpoints, and queue consumers.
- React + Vite for the SPA.
- Hono for Worker routing.
- D1 for small relational state.
- R2 for all large content, replayable artifacts, and streamed Agentic workflow traces.
- Queues for async jobs, retries, and decoupling slow or flaky work from requests.
- Cloudflare Access as the primary auth boundary.
- Zod schemas in `src/shared` for all client/worker/queue contracts.

## Storage Split

D1 is a small transactional database. Keep rows compact and queryable.

Store in D1:

- Internal UUIDs, ownership fields, relationships, status enums, timestamps.
- R2 object keys, content type, byte size, checksum, version, retention class.
- Job state, attempt count, last error summary, DLQ pointer.
- Agentic run lookup metadata: run ID, workflow version, status, failure summary, and R2 trace key.
- Audit event metadata.
- Small denormalized display fields that are needed for list views.

Do not store in D1 by default:

- PDFs, images, audio, video, zipped data, or generated files.
- Long transcripts, full model prompts/responses, large raw API payloads, full scraped pages.
- Unbounded JSON blobs.

Store these in R2 and keep D1 pointers. Exceptions require documented size limits, retention, and query reason.

## R2 Key Rules

Use stable, scoped, non-secret keys:

```text
{environment}/{tenant_or_user}/{domain}/{entity_id}/{version}/{filename}
```

Track `r2_key`, `content_type`, `byte_size`, `sha256`, `created_by`, and lifecycle status in D1. Keep originals immutable; new versions get new keys.

Agentic trace keys should make run lookup and lifecycle management straightforward:

```text
{environment}/{tenant_or_user}/agent-traces/{workflow}/{run_id}/{sequence_or_chunk}.jsonl
```

Write traces incrementally so failed runs preserve partial execution history. Store only non-secret lookup metadata in D1 and apply explicit retention/lifecycle rules.

## Queue Pattern

Use queues for slow, flaky, external, or retryable work.

1. Request validates input, writes D1 metadata, writes large bytes to R2, writes a job row, enqueues a message with IDs only, and returns quickly.
2. Consumer reads the job/entity IDs, rehydrates from D1/R2, performs work, updates D1 status, and records compact output metadata plus R2 keys for large output.
3. Retries must be idempotent. Use job status, attempt count, and unique operation keys to avoid duplicate side effects.
4. Configure a DLQ for production queues. A DLQ runbook is required before launch.

## Auth and Authorization

- Put production Workers behind Cloudflare Access.
- Verify the `Cf-Access-Jwt-Assertion` in Worker middleware.
- Normalize identity once and attach it to request context.
- Never trust client-supplied identity, role, email, tenant, or owner IDs.
- Repositories must include owner/tenant scope in SQL predicates.
- Services enforce role rules before calling repositories or adapters.

## Automatic Production Deploy

For small-team velocity, production deploys should happen automatically after GitHub checks pass.

Required guardrails:

- Branch protection for the production branch.
- Fast, hands-off CI tests.
- Typecheck, lint, migration validation, and Worker build.
- Smoke tests against a preview or local Miniflare environment where practical.
- Post-deploy synthetic checks for critical flows.
- Documented Cloudflare rollback path and a clear owner for responding to failed deploys.

Do not add a manual release board unless the risk justifies it. Prefer automation plus rollback.
