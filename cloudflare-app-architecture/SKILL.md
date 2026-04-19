---
name: cloudflare-app-architecture
description: "Design, review, and scaffold Cloudflare-first applications that should follow the Schovia single-Worker pattern: React SPA + Hono API + queue consumer in one deployable, D1/R2/Queues storage split, shared Zod contracts, explicit routes/services/repositories/adapters layering, and Cloudflare Access authentication. Use when creating a new Cloudflare app, evaluating whether an app matches this standard, planning project layout, defining storage and async job patterns, or implementing SSE-based LLM features on Cloudflare infrastructure."
---

# Cloudflare App Architecture

Follow this skill when the goal is to build or review a Schovia-style Cloudflare application. Treat consistency across sibling apps as a primary requirement, not an optimization target.

## Core Workflow

1. Confirm that the app fits the target shape: CRUD + async jobs + optional LLM features on Cloudflare infrastructure for a small-to-medium user base.
2. Keep the deployment topology to one Worker that serves the SPA, the JSON API, and the queue consumer.
3. Choose Cloudflare-native primitives first: Workers, D1, R2, Queues, KV, Durable Objects, and Vectorize before introducing external infrastructure.
4. Keep the client thin and the Worker authoritative. Derive identity server-side, validate every boundary with Zod, and enforce ownership in repositories and third-party filters.
5. Defer slow, flaky, or third-party work to Queues. Persist durable state in D1 and original bytes in R2 before enqueueing.
6. Preserve the explicit layer split: `routes -> services -> repositories -> adapters`. Avoid catchall utility layers and avoid burying SQL or external I/O outside their owning layers.

## Required Defaults

- Use TypeScript end-to-end.
- Use Hono for the Worker router.
- Use React + Vite for the SPA served through the Worker `ASSETS` binding.
- Use plain CSS unless the user gives a concrete reason to adopt something else.
- Use D1 for transactional state, R2 for original artifacts, and Queues for async handoff.
- Use shared Zod schemas and shared TS types from `src/shared`.
- Use SSE for incremental output instead of WebSockets unless a hard requirement says otherwise.

## Auth Standard

Put each Worker behind Cloudflare Access (Zero Trust).

- Treat Cloudflare Access as the default authentication boundary for Schovia apps.
- Configure Google as the IdP once at the Cloudflare account level.
- Assume Access sets a session cookie scoped to `schovia.work`, so auth on one subdomain can cover sibling apps on other `*.schovia.work` subdomains.
- Verify the `Cf-Access-Jwt-Assertion` header inside the Worker on every protected request.
- Reuse the Access identity in server-side authorization checks instead of trusting anything sent by the client.
- Add an application-level session only when the product needs extra app-specific state that Access does not carry.

## Project Shape

Keep this directory structure unless the task requires a narrow deviation:

```text
src/
  client/
  shared/
  worker/
migrations/
docs/
tests/
scripts/
wrangler.jsonc
package.json
tsconfig.json
vite.config.ts
```

Inside `src/worker`, preserve:

- `middleware/` for request context, auth, and other boundary middleware
- `routes/` for parsing, validation, and delegation only
- `services/` for multi-step business workflows
- `repositories/` for D1 SQL
- `adapters/` for external I/O
- `lib/` for Worker-only support helpers

## Decision Rules

- Reject proposals that split the app into multiple deployables unless there is a concrete operational requirement.
- Reject requests to let the client send `user_id` or to perform authorization client-side.
- Reject designs that store critical state only in a third-party provider.
- Prefer plain SQL repositories over introducing an ORM.
- Prefer REST + SSE over GraphQL and WebSockets for this stack.
- Call out deviations explicitly, including why the default pattern is insufficient.

## Reference

Read [references/architecture-design.md](./references/architecture-design.md) when you need the full standard, including stack rationale, runtime topology, storage split, async job lifecycle, deployment conventions, and the detailed Cloudflare Access guidance.
