# manifest-backend

NestJS 11 API server. Serves the REST API, the LLM proxy, and (in production) the frontend static files from one port.

## Module Map

| Module | Page | Responsibility |
|--------|------|----------------|
| `main.ts` | [main.md](main.md) | Bootstrap: Helmet, CORS, Better Auth mount, server start |
| `app.module.ts` | [main.md](main.md) | Root module — imports all features, global guards/cache |
| `auth/` | [auth.md](auth.md) | Better Auth session guard, social OAuth, `@CurrentUser()` decorator |
| `database/` | [database.md](database.md) | TypeORM config, migrations, seeder, OpenRouter pricing sync, Ollama sync |
| `entities/` | [entities.md](entities.md) | All 23 TypeORM entities |
| `common/` | [common.md](common.md) | Guards, decorators, interceptors, utilities, constants |
| `health/` | [health.md](health.md) | `GET /api/v1/health` — always 200 |
| `analytics/` | [analytics.md](analytics.md) | Dashboard data: overview, tokens, costs, messages, agents, savings |
| `otlp/` | [otlp.md](otlp.md) | Agent API key auth guard, agent onboarding service |
| `routing/` | [routing.md](routing.md) | LLM proxy, tier/specificity/provider config, model discovery hook-in |
| `scoring/` | [scoring.md](scoring.md) | 30-dimension complexity scorer, specificity detector |
| `model-discovery/` | [model-discovery.md](model-discovery.md) | Provider model fetching, enrichment, OpenRouter fallback |
| `model-prices/` | [model-prices.md](model-prices.md) | Model pricing cache service and controller |
| `free-models/` | [free-models.md](free-models.md) | Free/zero-cost model aggregation and sync |
| `notifications/` | [notifications.md](notifications.md) | Alert rules, threshold cron, email dispatch |
| `public-stats/` | [public-stats.md](public-stats.md) | Unauthenticated aggregate stats endpoint |
| `setup/` | [setup.md](setup.md) | First-run admin account creation |
| `github/` | [github.md](github.md) | GitHub star count proxy |
| `sse/` | [sse.md](sse.md) | Server-Sent Events for real-time dashboard updates |
| `telemetry/` | [telemetry.md](telemetry.md) | Anonymous self-hosted telemetry |

## Key Dependencies

- `manifest-shared` — canonical provider/tier/specificity types
- `better-auth` — user session management
- `typeorm` + `pg` — PostgreSQL ORM
- `@nestjs/schedule` — cron jobs (pricing sync, notification check, telemetry)
- `@nestjs/cache-manager` — in-memory caching (routing, analytics)
- `helmet` — security headers
- `express-rate-limit` — per-endpoint rate limiting (auth routes, proxy)

## Build

```bash
cd packages/backend
npx nest build           # compile to dist/
node -r dotenv/config dist/main.js   # run
```

## See Also

- [../../architecture.md](../../architecture.md) — system topology
- [../../development.md](../../development.md) — dev workflow
