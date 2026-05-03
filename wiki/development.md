# Development

Local development setup, conventions, and workflows.

## Quick Start

### 1. Start PostgreSQL

```bash
docker start postgres_db 2>/dev/null || \
  docker run -d --name postgres_db \
    -e POSTGRES_USER=myuser -e POSTGRES_PASSWORD=mypassword \
    -e POSTGRES_DB=mydatabase -p 5432:5432 postgres:16
```

Always use a **fresh uniquely-named database** to avoid cross-session data pollution:

```bash
DB_NAME="manifest_$(openssl rand -hex 4)"
docker exec postgres_db psql -U myuser -d postgres -c "CREATE DATABASE $DB_NAME;"
```

### 2. Configure `.env`

Minimal `packages/backend/.env`:

```env
PORT=3001
BIND_ADDRESS=127.0.0.1
NODE_ENV=development
BETTER_AUTH_SECRET=<output of: openssl rand -hex 32>
DATABASE_URL=postgresql://myuser:mypassword@localhost:5432/$DB_NAME
API_KEY=dev-api-key-12345
SEED_DATA=true
```

### 3. Start Dev Servers (parallel)

```bash
# Backend — must preload dotenv (auth.instance.ts reads process.env at import time)
cd packages/backend && NODE_OPTIONS='-r dotenv/config' npx nest start --watch

# Frontend
cd packages/frontend && npx vite
```

`npm run dev` (turbo) only starts the frontend — start the backend separately as above.

## Seed Data

With `SEED_DATA=true`:

| Resource | Value |
|----------|-------|
| Admin user | `admin@manifest.build` / `manifest` |
| Tenant | `seed-tenant-001` |
| Agent | `demo-agent` |
| OTLP key | `dev-otlp-key-001` |
| API key | `dev-api-key-manifest-001` |
| Messages | Sample telemetry for demo-agent |

Seeding is idempotent.

## Tests

```bash
npm test --workspace=packages/backend           # Jest unit tests
npm run test:e2e --workspace=packages/backend   # Jest E2E (requires PostgreSQL)
npm test --workspace=packages/frontend          # Vitest
npm test --workspace=packages/shared            # Jest
```

### Coverage (100% required)

Every PR must maintain 100% line coverage across all packages. Run before opening a PR:

```bash
cd packages/backend && npx jest --coverage
cd packages/frontend && npx vitest run --coverage
cd packages/shared && npx jest --coverage
```

### E2E Test Entities

When adding a new TypeORM entity to `database.module.ts`, also add it to `packages/backend/test/helpers.ts` entities array. Missing entities cause `EntityMetadataNotFoundError` in E2E tests.

## Database Migrations

Schema sync (`synchronize`) is **permanently disabled**. All changes go through migrations.

```bash
cd packages/backend

# After modifying an entity:
npm run migration:generate -- src/database/migrations/DescriptiveName

# Other commands:
npm run migration:run      # Run pending
npm run migration:revert   # Revert last
npm run migration:show     # Show status ([X] = applied)
npm run migration:create -- src/database/migrations/EmptyName
```

After generating, import the migration in `database.module.ts` and add to the `migrations` array. Always use unique timestamps — never reuse an existing timestamp.

## Shared Projection Contract {#shared-projection}

Any endpoint that returns rows rendered by `MessageTable` / `ModelCell` **must** use `selectMessageRowColumns()` from `analytics/services/query-helpers.ts`. Never hand-roll a SELECT chain for these endpoints. The helper is pinned by `query-helpers.spec.ts` which fails loudly if a required column is dropped.

## Adding a New Provider

1. Add `SharedProviderEntry` to `SHARED_PROVIDERS` in `packages/shared/src/providers.ts`
2. Add `FetcherConfig` in `packages/backend/src/model-discovery/provider-model-fetcher.service.ts`
3. Add `ProviderEndpoint` in `packages/backend/src/routing/proxy/provider-endpoints.ts`
4. (Optional) Add UI bits in `packages/frontend/src/services/providers.ts`

## Adding a New Specificity Category

1. Add ID to `SPECIFICITY_CATEGORIES` in `packages/shared/src/specificity.ts`
2. Add keyword file at `packages/backend/src/scoring/keywords/{category}.ts`
3. Add dimension to `DEFAULT_CONFIG.dimensions` in `scoring/config.ts`
4. Add to `DIMENSION_MAP` in `scoring/specificity-detector.ts`
5. Optionally add to `TOOL_NAME_PATTERNS`
6. Add `StageDef` to `packages/frontend/src/services/providers.ts`
7. Add test prompts to `scoring/__tests__/specificity-coverage.spec.ts`

## Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `BETTER_AUTH_SECRET` | — | **Required.** ≥32 chars. `openssl rand -hex 32` |
| `DATABASE_URL` | `postgresql://myuser:mypassword@localhost:5432/mydatabase` | PostgreSQL connection |
| `PORT` | `3001` | Server port |
| `BIND_ADDRESS` | `127.0.0.1` | Use `0.0.0.0` for Docker/Railway |
| `NODE_ENV` | — | `development` enables CORS |
| `CORS_ORIGIN` | `http://localhost:3000` | Allowed CORS origin |
| `BETTER_AUTH_URL` | `http://localhost:{PORT}` | Base URL for Better Auth callbacks |
| `FRONTEND_PORT` | — | Extra trusted origin port |
| `API_KEY` | — | `X-API-Key` header secret |
| `THROTTLE_TTL` | `60000` | Rate limit window (ms) |
| `THROTTLE_LIMIT` | `100` | Max requests per window |
| `MAILGUN_API_KEY` | — | Email verification/password reset |
| `MAILGUN_DOMAIN` | — | Mailgun sending domain |
| `NOTIFICATION_FROM_EMAIL` | `noreply@manifest.build` | Sender address |
| `GOOGLE_CLIENT_ID/SECRET` | — | Google OAuth |
| `GITHUB_CLIENT_ID/SECRET` | — | GitHub OAuth |
| `DISCORD_CLIENT_ID/SECRET` | — | Discord OAuth |
| `SEED_DATA` | — | `true` to seed demo data |
| `MANIFEST_MODE` | `cloud` | `cloud` or `selfhosted` |
| `OLLAMA_HOST` | `http://localhost:11434` | Ollama endpoint |
| `MANIFEST_TELEMETRY_DISABLED` | — | `1` to disable telemetry |

## Content Security Policy

Helmet enforces a strict CSP that only allows `'self'` origins. **Never load external CDN resources.** All fonts, icons, and assets must be self-hosted in `packages/frontend/public/`.

## See Also

- [architecture.md](architecture.md) — system topology
- [deployment.md](deployment.md) — Docker and CI/CD
- [entities.md](entities.md) — migration guidance
- [providers.md](providers.md) — adding providers
- [specificity.md](specificity.md) — adding categories
