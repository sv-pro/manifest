# Architecture

Manifest is a single-service, monorepo application: one NestJS process serves both the REST API and the pre-built SolidJS frontend from the same port.

## Monorepo Layout

```
packages/
├── shared/    # manifest-shared  — canonical types shared by backend + frontend
├── backend/   # manifest-backend — NestJS API + proxy + scheduler
├── frontend/  # manifest-frontend — SolidJS SPA
└── manifest/  # manifest          — version shell (no code; holds Docker version)
```

Tooling: **npm workspaces** for dependency hoisting, **Turborepo** for build orchestration. `npm run build` compiles frontend (Vite) then backend (Nest) in dependency order.

## Tech Stack

| Layer | Technology |
|-------|-----------|
| API server | NestJS 11 (Express platform) |
| ORM | TypeORM 0.3 |
| Database | PostgreSQL 16 |
| Auth | Better Auth 1.x |
| Input validation | class-validator + class-transformer |
| Security headers | Helmet |
| Rate limiting | @nestjs/throttler + express-rate-limit (auth endpoints) |
| Frontend | SolidJS 1.9 |
| Build tool | Vite 6.1 |
| Charts | uPlot 1.6 |
| Email templates | React Email |
| Runtime | Node.js 24.x, TypeScript 5.7 (strict) |

## Request Entry Points

| Path | Handler | Auth |
|------|---------|------|
| `POST /v1/chat/completions` | ProxyController | Agent API key (mnfst_*) |
| `POST /v1/responses` | ProxyController | Agent API key (mnfst_*) |
| `ALL /api/auth/*` | Better Auth middleware | Public |
| `GET /api/v1/*` | NestJS controllers | Session or X-API-Key |
| `GET /api/v1/events` | SSE controller | Session |

## Single-Service Deployment

In production, `@nestjs/serve-static` serves `packages/frontend/dist/` with SPA fallback. `/api/*` and `/otlp/*` are excluded from static serving so API routes take priority.

```
Internet → [single port] → NestJS
                            ├── /api/*       → NestJS controllers
                            ├── /v1/*        → Proxy (agent-key auth)
                            └── /*           → Static frontend files
```

## Dev Mode

Vite dev server runs on `:3000` and proxies `/api` and `/otlp` to the NestJS backend on `:3001`. CORS is enabled only in `NODE_ENV=development`.

```
Browser → Vite :3000 → /api/* → NestJS :3001
                     → /*     → Vite HMR
```

## Body Parsing

NestJS body parsing is **disabled** at the framework level (`bodyParser: false`). Better Auth is mounted first as raw Express middleware (needs body control), then `express.json()` and `express.urlencoded()` are added for all other routes.

## Configuration

Environment is loaded via `packages/backend/.env`. `auth.instance.ts` reads `process.env` at import time (before NestJS ConfigModule loads), so env vars must be present in the process before the app starts — hence `NODE_OPTIONS='-r dotenv/config'` in dev.

## See Also

- [data-flow.md](data-flow.md) — request lifecycle
- [authentication.md](authentication.md) — guard chain
- [deployment.md](deployment.md) — Docker, Railway, CI/CD
- [development.md](development.md) — local dev setup
- [code/backend/index.md](code/backend/index.md) — backend module map
