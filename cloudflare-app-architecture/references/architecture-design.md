# Architecture & Design

This reference captures the architecture, design philosophy, and stack decisions used in `customGPT` so that sibling applications can be built with a consistent shape. The specifics here come from that codebase, but the intent is to describe the pattern: sibling apps should adopt the same stack, directory shape, and cross-cutting conventions even when the domain is different.

## 1. Design Philosophy

The system is built around a small set of principles. Sibling applications should honor the same ones.

1. **One Worker, one deployable.** The same Cloudflare Worker serves the static SPA, the JSON API, and the queue consumer. There is no separate backend service, API gateway, or jobs worker. One `wrangler deploy` ships everything. Favor fewer moving parts, atomic rollbacks, and no cross-service auth.
2. **Edge-native, serverless-first.** Default to Cloudflare primitives (Workers, D1, R2, Queues, KV, Durable Objects, Vectorize) before reaching for anything else. They share a deployment model, a secrets model, and a billing surface. Avoid long-lived servers, container orchestration, and regional infrastructure unless a requirement forces it.
3. **Thin client, authoritative server.** The client is a render layer. It never sends `user_id`, never computes authorization, and never trusts its own state. The server derives identity from Cloudflare Access on every request and re-validates ownership on every read and write.
4. **Defer expensive work to a queue.** Any operation that is slow, flaky, calls a third party, or must survive a page close goes through Cloudflare Queues. The synchronous path stays fast and idempotent; the async path handles retries and partial failure.
5. **Store originals in R2, state in D1, and nothing critical in the LLM.** The durable source of truth is always local storage. Third-party services (OpenAI, vector stores, external APIs) are rebuildable caches. Keep the inputs so re-ingestion is always possible.
6. **Typed end-to-end, validated at the boundary.** TypeScript everywhere. Zod schemas at every HTTP and queue boundary. `src/shared` holds types used by both client and worker so the contract cannot drift.
7. **Small, boring dependencies.** Hono for routing, Zod for validation, the provider SDK for the LLM, and the framework for the UI. No ORM, no state-management library, no CSS framework, no dependency injection container. If a pattern can be expressed in about 50 lines of TypeScript, keep it simple.
8. **Explicit layers, no clever abstractions.** `routes -> services -> repositories -> adapters`. Each layer has one job. Repositories own SQL. Adapters own external I/O. Services compose them. Routes parse and validate. Avoid manager, helper, or util catchalls.

## 2. Stack

| Layer | Choice | Why |
| --- | --- | --- |
| Compute | **Cloudflare Workers** | Global edge, millisecond cold starts, single-binary deploy, bindings for storage primitives. |
| HTTP router | **Hono** | Minimal, Workers-native, good TypeScript ergonomics, composable middleware. |
| Relational store | **Cloudflare D1** (SQLite) | Bound directly to the Worker, no connection pool, no network egress. Fine for read-heavy plus small-to-medium write volume. |
| Object store | **Cloudflare R2** | S3-compatible, zero egress fees, direct Worker binding. Holds every original uploaded artifact. |
| Async jobs | **Cloudflare Queues** | Native producer plus consumer in the same Worker, automatic retries, dead-letter support. |
| Validation | **Zod** | Schema is the type. Runtime validation and compile-time types from one declaration. Shared between client and worker. |
| Frontend | **React 19 + Vite** | SPA built to static assets, served by the same Worker via the `ASSETS` binding. |
| Styling | Plain CSS (`styles.css`) | No Tailwind, no CSS-in-JS. Keeps build output small. |
| LLM / retrieval | **OpenAI Responses API + hosted vector store** | Avoid running a custom embedding pipeline or vector DB. Tenant isolation stays enforced with metadata filters. |
| Auth | **Cloudflare Access (Zero Trust)** | Shared account-level Google IdP, cross-subdomain session coverage on `schovia.work`, signed JWT assertion per request, no custom public auth flow in the hot path. |
| Tests | **Vitest** | Same runner for worker and client code. |
| Deploy | **Wrangler** | Single command. Secrets, bindings, and migrations are declared next to the code. |

Do not substitute casually. A sibling app can pick a different LLM provider, but it should not swap Hono for Express, D1 for Postgres, or React for Svelte without a concrete reason. Consistency across the fleet is worth more than local optimization.

## 3. Runtime Topology

```text
         browser
  React SPA (served by ASSETS binding)
              |
              | fetch /api/*   (SSE for chat)
              v
     Cloudflare Worker
       Hono router
        - middleware (requestContext, requireAccessAuth)
        - routes/        (parse + validate)
        - services/      (business logic)
        - repositories/  (D1 SQL)
        - adapters/      (OpenAI, Access JWT verify, R2)
       Queue consumer (same entrypoint)
          |        |        |        |
          v        v        v        v
         D1       R2      Queue    OpenAI
```

Key properties:

- **Single Worker entrypoint** (`src/worker/index.ts`) exports both `fetch` (HTTP) and `queue` (consumer) handlers.
- **Static assets** are served by the Workers `ASSETS` binding from `./dist/client` with `run_worker_first: true`, so API routes take precedence and everything else falls through to the SPA.
- **No separate origin, no separate CDN.** The Worker is the origin and the edge.
- **Each app sits behind Cloudflare Access.** Identity arrives at the Worker through the signed `Cf-Access-Jwt-Assertion` header after Access authenticates the browser.

## 4. Request Lifecycle

Every protected HTTP request follows the same path:

1. **`requestContext` middleware** assigns a `requestId` for correlation.
2. **`requireAccessAuth` middleware** reads `Cf-Access-Jwt-Assertion`, verifies the JWT signature, issuer, audience, and expiry, and attaches an authenticated user object to context. A missing or invalid assertion returns `401` immediately.
3. **Route handler** parses the body or form-data, validates with a Zod schema, and extracts path/query params. Routes do not contain business logic beyond assembling inputs.
4. **Service** performs the operation: ownership checks, DB writes, R2 writes, queue enqueue, adapter calls. Services are the only place where a multi-step workflow is expressed.
5. **Repository** executes SQL against D1. Every query takes the owner id (and usually the scoping entity id) as a parameter. There is no get-by-id that does not also filter by owner.
6. **Adapter** talks to anything outside the Worker: OpenAI, Cloudflare Access token verification helpers, external APIs, R2 helpers, crypto helpers. Adapters return plain data, not SDK shapes, so services are easy to test.
7. **Response** is JSON (via helpers such as `jsonOk` / `jsonError`) or an SSE stream for streaming endpoints. Errors throw `HTTPException` and are rendered by a global error handler.

Chat is the main exception: it returns a `text/event-stream` response whose body is a `ReadableStream` produced by the chat service wrapping the provider SDK's async iterator.

## 5. Data Ownership Model

This is the most important invariant in the system. Sibling apps should copy it verbatim.

- **The client never sends `user_id`.** Not in the body, not in a header, not in a query param.
- **Identity comes from Cloudflare Access**, resolved server-side after verifying `Cf-Access-Jwt-Assertion`.
- **Every repository method takes the owner id as a parameter** and includes it in the `WHERE` clause. There is no `getGptById(id)`; there is only `getGpt(userId, id)`.
- **Nested resources are double-scoped.** A knowledge file is looked up by `(user_id, gpt_id, file_id)`. A message is looked up by `(user_id, conversation_id, message_id)`.
- **Third-party retrieval is also scoped.** When calling a shared vector store, every query includes `user_id` and the relevant parent entity id as metadata filters. The shared store is treated as untrusted. Isolation is enforced in application logic, not delegated to the vendor.
- **Soft-delete before hard-delete** for anything a user can recover from, or anything tied to an external resource whose deletion might fail.

A sibling app with different domain entities should still follow this pattern: never accept owner identity from the client, always scope at the SQL layer, and always filter at the third-party layer.

## 6. Storage Split

Each store has one job. Do not blur them.

- **D1**: transactional state such as users, entities, relationships, status fields, and audit timestamps. Keep rows small. Give every table `id` (text UUID), `user_id`, `created_at`, and `updated_at` where relevant. Declare foreign keys, but enforce isolation in the query shape, not by relying on `PRAGMA foreign_keys`.
- **R2**: original user-supplied bytes such as uploads, exports, and generated artifacts. Key by a hierarchical path such as `{user_slug}/{parent_id}/{entity_id}{ext}`. Do not mutate originals; new versions get new keys.
- **Queues**: ephemeral job handoff. Messages carry only the minimal identifier, never the full payload. The consumer re-reads state from D1 and bytes from R2. This keeps every job idempotent and retry-safe.
- **Third-party services**: rebuildable. If the vendor lost all app data tomorrow, the app could re-ingest from R2 plus D1.

Never store a third-party id as the primary key. Keep an internal UUID as the primary key and store vendor ids as nullable columns that become populated when indexing succeeds.

## 7. Async Job Pattern

For anything slow, flaky, or external:

1. **Synchronous path** validates input, writes the durable record with `status: "pending"`, writes original bytes to R2, writes a job row, enqueues a message containing only the entity id, and returns `201`.
2. **Queue consumer** reads the id, rehydrates from D1 plus R2, calls the external service, updates `status` to `ready` or `failed`, and stores any provider-assigned ids. Errors bump an `attempt_count` and are surfaced on the record.
3. **Client** observes status by re-reading the parent entity. Long-running UIs should poll or subscribe via SSE; they must not assume readiness from the initial `201`.
4. **Cleanup** on delete reverses the external operation (delete from provider, delete from R2) before marking the DB row deleted.

Job rows (`*_jobs` tables) stay separate from entity rows so retry metadata does not pollute the domain model.

## 8. Streaming

Any endpoint producing incremental output (LLM responses, long reports, progress) returns `text/event-stream`. Conventions:

- Event envelopes are JSON, one per `data:` line, with a discriminated-union `type` field.
- The Worker wraps the upstream SDK's async iterator in a `ReadableStream` and writes events as they arrive.
- The client accumulates text on `*.delta`, finalizes on `*.completed`, and surfaces errors on `*.error`.
- The server writes the terminal record to D1 after the stream closes, not during it, so a disconnected client does not corrupt state.

This avoids WebSockets entirely. SSE works through common proxies, requires no special client, and fits cleanly into `fetch`.

## 9. Auth

- **Cloudflare Access is the standard auth boundary.** Put each Worker behind Access and let Access handle browser sign-in before requests hit the app.
- **Configure Google as the IdP once at the account level.** Reuse it across Schovia apps instead of wiring separate OAuth flows per Worker.
- **Rely on the shared Access session cookie scoped to `schovia.work`.** Signing in at `a.schovia.work` should also cover `b.schovia.work` as long as Access policy allows it.
- **Verify `Cf-Access-Jwt-Assertion` on every protected request.** Validate the signature against Cloudflare's public keys and verify audience, issuer, and expiry before trusting identity claims.
- **Do not trust client-provided identity.** Authorization still happens in repositories and services using the verified subject from Access.
- **Only add application-level session state when needed.** If the app must remember product-specific onboarding state or preferences, store that separately. Do not replace Access with a parallel auth system in the hot path.
- **Keep Access verification inside adapters or auth middleware.** Routes should consume an authenticated user object, not parse JWTs directly.

Sibling apps should reuse the same adapter shape: one module verifies Access assertions and returns a normalized identity object for downstream services.

## 10. Project Layout

```text
<app-root>/
├── src/
│   ├── client/             React SPA. Mounted by main.tsx.
│   │   ├── App.tsx         Top-level view state machine.
│   │   ├── components/     Presentational and small stateful components.
│   │   ├── lib/            Client-only helpers (fetch wrappers, formatters).
│   │   ├── main.tsx
│   │   └── styles.css
│   ├── shared/             Code imported by both client and worker.
│   │   ├── schema.ts       Zod schemas for request/response contracts.
│   │   ├── types.ts        TypeScript types inferred from schemas or declared.
│   │   └── <domain>.ts     Pure domain constants (enums, allowlists).
│   └── worker/
│       ├── index.ts        fetch + queue entrypoint.
│       ├── middleware/     requestContext, requireAccessAuth, etc.
│       ├── routes/         One file per resource. Parse, validate, delegate.
│       ├── services/       Multi-step business logic.
│       ├── repositories/   D1 SQL. One repository per bounded context.
│       ├── adapters/       External I/O: Access JWT verify, provider SDKs, R2.
│       └── lib/            Worker-only helpers (http, crypto primitives).
├── migrations/             Ordered SQL files: 0001_init.sql, 0002_*.sql, ...
├── docs/
│   ├── Product.md
│   ├── Engineering.md
│   ├── ArchitectureDesign.md
│   ├── Deploy.md
│   ├── DeploySecrets.md
│   └── Troubleshooting.md
├── tests/                  Vitest specs. Mirror `src/` layout.
├── scripts/                Operational scripts (seed, dev helpers, etc.).
├── wrangler.jsonc          Bindings, compat date, assets, observability.
├── package.json            Lean dependencies only.
├── tsconfig.json           Strict mode on.
└── vite.config.ts
```

The `src/shared` directory is the contract between client and worker. If a type or constant is used by both, put it there and nowhere else.

## 11. Cross-Cutting Conventions

- **IDs**: use `crypto.randomUUID()` text primary keys. No auto-increment.
- **Timestamps**: use ISO 8601 strings (`new Date().toISOString()`), stored as `TEXT`. Avoid SQLite's implicit `CURRENT_TIMESTAMP` for portability.
- **Status enums**: use string literals defined in `src/shared`, validated by Zod, and stored as `TEXT`. Do not create lookup tables for simple status sets.
- **Errors**: routes throw `HTTPException`; adapters throw typed errors that services translate; services never leak provider error shapes to clients.
- **Logging**: emit one structured line per request including `requestId`, route, status, and duration. Avoid unstructured `console.log`.
- **Secrets**: declare them in `docs/DeploySecrets.md`, set them via `wrangler secret put`, and read them from `env.*` only inside adapters.
- **Migrations**: use forward-only numbered SQL files. Run pending migrations before the Worker version cuts over. Never edit an applied migration.
- **Validation**: keep one Zod schema per boundary shape. Parse at the boundary and pass typed objects downstream.
- **Testing**: unit test schemas, crypto helpers, Access assertion verification helpers, and services. Integration test route handlers with Miniflare. Add end-to-end streaming tests for chat-like endpoints.

## 12. What Is Deliberately Absent

These are common choices that are not part of this stack. A sibling app should justify introducing any of them.

- **ORMs (Prisma, Drizzle)**: repositories write plain SQL. Query readability beats schema abstraction at this scale.
- **Redis or an external cache**: D1 and the Workers runtime cover the baseline needs. If caching is required, prefer the Cache API or KV.
- **Message brokers beyond Cloudflare Queues**: no SQS, RabbitMQ, or Kafka by default.
- **Separate backend-for-frontend**: the Worker is both backend and frontend origin.
- **State-management libraries**: React state plus URL state is enough until the view proves otherwise.
- **Component libraries**: plain components in `src/client/components`.
- **GraphQL**: use REST plus SSE. Type safety comes from shared Zod schemas.
- **Docker, Kubernetes, or VMs**: default to the Workers runtime only.
- **Runtime dependency injection**: prefer explicit imports.

## 13. Deployment Model

- `wrangler.jsonc` declares every binding: D1 database, R2 bucket, queue producer plus consumer, assets directory, and any other Worker binding. Bindings are the only environment-specific surface.
- Secrets are out-of-band (`wrangler secret put`) and never committed to source control.
- `npm run build` produces `dist/client/`; `wrangler deploy` uploads the Worker and the static assets in one atomic version.
- Rollback is `wrangler rollback` or the Deployments UI. The whole app (API, UI, queue consumer) rolls back together.
- Observability is enabled in `wrangler.jsonc`; invocation logs are the default telemetry surface.

Sibling apps should preserve the same single-deployable invariant. If a feature appears to require a second Worker or a separate service, reconsider the design first.

## 14. When To Deviate

This architecture targets applications that are CRUD plus async jobs plus LLM interaction for small-to-medium user bases at the edge. It fits content tools, authoring tools, internal assistants, team-sized SaaS, and vertical AI apps.

It is not the right fit for every workload. Raise the design question explicitly when the app needs:

- High-write OLTP at thousands of writes per second: D1 is the wrong store.
- Multi-region strongly consistent data: use Durable Objects or a dedicated database.
- Long-running compute above Worker CPU limits: move compute elsewhere.
- Heavy binary processing in-request: move it to a dedicated service or Cloudflare Containers.
- Regulated data with residency constraints that the edge model cannot satisfy.

In most other cases, the expectation is the same stack, the same layout, and the same conventions. Consistency is the point.
