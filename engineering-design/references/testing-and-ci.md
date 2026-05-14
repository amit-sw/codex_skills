# Testing and CI Reference

Use this reference when defining verification and GitHub deployment gates.

## Minimum Test Set

Required for mission-critical Cloudflare services:

- TypeScript typecheck.
- Lint or static checks matching repo conventions.
- Unit tests for pure logic, schemas, services, and adapters.
- Repository tests for D1 SQL against Miniflare or a local D1-compatible harness.
- Route tests for auth, validation, success, and common failure paths.
- Queue consumer tests for idempotency, retry, poison message, and DLQ behavior.
- R2 adapter tests for key generation, metadata, missing object, and content-type handling.
- Access auth middleware tests for missing, invalid, expired, and valid JWT cases.
- Admin audit tests proving both actor and effective user are recorded.
- LLM workflow tests with mocked providers, schema validation, trace streaming to R2, failed-run partial trace preservation, and replay fixtures.
- E2E or smoke tests for the most critical user flows.

Coverage targets are useful but insufficient. A small service can have high coverage and still miss queue retries, migration safety, or auth bypasses.

## GitHub Checks for Auto-Deploy

Automatic production deploy requires these checks to pass:

```text
npm ci
npm run typecheck
npm run lint
npm test
npm run build
wrangler deploy --dry-run or equivalent validation
```

Adapt command names to the repository, but keep the gate categories.

If migrations exist, CI must validate:

- Migration files are forward-only and ordered.
- Previously applied migrations were not edited.
- Fresh database creation works.
- Current schema plus new migrations works.
- Rollback plan is documented even if migrations are forward-only.

## Test Data Rules

- Fixtures must be committed and deterministic.
- Never use production secrets in tests.
- Use small R2 fixture objects unless testing size behavior.
- Include fixtures for failure paths, malformed input, unauthorized access, and partial completion.

## Release Confidence for Small Teams

Avoid heavyweight manual QA as the default. Instead require:

- A fast local test command.
- The same tests in CI.
- Preview or local smoke checks.
- Post-deploy synthetic checks.
- Clear rollback instructions.

When a change touches auth, audit, storage migration, queue processing, or LLM tool permissions, add targeted tests before deploy.

When a change touches an Agentic workflow, add targeted tests proving trace events are written incrementally to R2, D1 lookup metadata points to the trace, and a failed run can be replayed from the preserved trace.
