# Manifest Wiki

> Persistent knowledge base for the Manifest codebase. Maintained by LLMs following the [Karpathy wiki pattern](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f) — the LLM writes and updates pages; humans read and direct.

See [schema.md](schema.md) for conventions and workflows. See [log.md](log.md) for change history.

---

## Overview Pages

| Page | Summary |
|------|---------|
| [architecture.md](architecture.md) | System topology: single-service NestJS + SolidJS monorepo, tech stack, deployment model |
| [data-flow.md](data-flow.md) | End-to-end request lifecycle from proxy entry to DB persistence |
| [authentication.md](authentication.md) | Three-guard chain (Session → ApiKey → Throttler), Better Auth, agent API keys |
| [multi-tenancy.md](multi-tenancy.md) | User → Tenant → Agent → AgentApiKey ownership model and data isolation |
| [routing.md](routing.md) | Two-layer LLM routing: complexity tiers + specificity categories, fallback chains |
| [scoring.md](scoring.md) | 30-dimension complexity scoring engine that assigns simple/standard/complex/reasoning |
| [specificity.md](specificity.md) | 9 task-type categories (coding, trading, …), detection algorithm, session bias |
| [providers.md](providers.md) | Canonical provider registry (17 providers), encryption, local-only vs. cloud |
| [models.md](models.md) | Model discovery, OpenRouter pricing cache, quality scoring, tier auto-assignment |
| [entities.md](entities.md) | All 23 TypeORM entities with roles and key columns |
| [api.md](api.md) | Complete HTTP endpoint reference (55+ routes) |
| [notifications.md](notifications.md) | Alert rules, threshold checking, Mailgun/Resend/SMTP email dispatch |
| [telemetry.md](telemetry.md) | Anonymous self-hosted telemetry: payload fields, cadence, opt-out |
| [deployment.md](deployment.md) | Docker image, changesets, CI/CD triggers, Railway support |
| [development.md](development.md) | Dev server, seeding, migrations, tests, 100% coverage requirement |

---

## Code Wiki

Hierarchical summaries of every package and module. Each package has an `index.md`; each module has its own page.

| Path | Summary |
|------|---------|
| [code/index.md](code/index.md) | Code wiki root — package map and cross-package conventions |
| [code/shared/index.md](code/shared/index.md) | `manifest-shared` — canonical types consumed by both backend and frontend |
| [code/backend/index.md](code/backend/index.md) | `manifest-backend` — NestJS API server with 18 top-level modules |
| [code/frontend/index.md](code/frontend/index.md) | `manifest-frontend` — SolidJS SPA dashboard |
| [code/manifest/index.md](code/manifest/index.md) | `manifest` — code-free shell holding the canonical Docker image version |
