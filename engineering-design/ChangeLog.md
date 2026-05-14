# Change Log

## 2026-05-14

- Added a hard requirement that all Agentic workflows stream execution traces to R2 during the run, with D1 metadata pointers for immediate internal replay after reported failures.
- Updated Cloudflare runtime, LLM, testing, observability, operations, and production-readiness guidance to require trace retention, redaction, replay access controls, alerting, runbooks, and tests.
- Added this changelog and a skill maintenance rule requiring major skill changes to be recorded here.
- Added a requirement to explain ramifications and get explicit user confirmation before changes with significant architecture, future maintainability, or observability impact.
- Added a requirement to avoid GitHub Actions for CI/CD and rely on Cloudflare Workers' GitHub integration for automatic build/deploy workflows.
