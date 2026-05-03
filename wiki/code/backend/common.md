# backend/common/

Shared guards, utilities, interceptors, and constants used across all backend modules.

## Guards

| File | Description |
|------|-------------|
| `guards/api-key.guard.ts` | Checks `X-API-Key` header against `API_KEY` env var using `crypto.timingSafeEqual`. Falls through if session guard already set. |

## Decorators

| File | Description |
|------|-------------|
| `decorators/public.decorator.ts` | `@Public()` — marks a route as unauthenticated. Skips SessionGuard and ApiKeyGuard. |

## Filters

| File | Description |
|------|-------------|
| `filters/spa-fallback.filter.ts` | In production, serves `index.html` for any non-API 404 (SPA client-side routing). |

## Interceptors

| File | Description |
|------|-------------|
| `interceptors/agent-cache.interceptor.ts` | Caches agent resolution by `(userId, agentName)`. |
| `interceptors/user-cache.interceptor.ts` | Caches user session data for repeated requests. |

## Services

| File | Description |
|------|-------------|
| `services/ingest-event-bus.ts` | In-process event bus for SSE — broadcasts agent_message inserts to connected dashboard clients. |
| `services/manifest-runtime.ts` | Runtime helpers: detect Docker (`/.dockerenv`), resolve `MANIFEST_MODE`. |
| `services/tenant-cache.ts` | Caches tenant lookups by `userId`. |

## Constants

| File | Description |
|------|-------------|
| `constants/providers.ts` | Re-exports `SHARED_PROVIDERS` under backend-facing names. See [../shared/providers.md](../shared/providers.md). |
| `constants/cache.constants.ts` | TTL values for all cache tiers (routing, analytics, public stats). |
| `constants/ollama.constants.ts` | Ollama default host and sync interval. |
| `constants/api-key.constants.ts` | `mnfst_` prefix constant. |

## Utilities

| File | Description |
|------|-------------|
| `utils/hash.util.ts` | `hashApiKey(key)` — scrypt KDF, used for agent API key storage |
| `utils/crypto.util.ts` | `encrypt(plain, secret)` / `decrypt(cipher, secret)` — AES-256-GCM |
| `utils/postgres-sql.ts` | TypeORM query builder helpers: `bucketColumn`, `castInterval`, `toCharDate` |
| `utils/slugify.ts` | Converts agent names to URL-safe slugs |
| `utils/url-validation.ts` | Validates provider base URLs; rejects private IPs in cloud mode |
| `utils/provider-inference.ts` | Infers `providerId` from a raw model name string |
| `utils/range.util.ts` | Parses `?range=7d` query param into a SQL interval |
| `utils/period.util.ts` | Time period helpers (start/end of day/week/month) |

## See Also

- [auth.md](auth.md) — SessionGuard (companion guard)
- [otlp.md](otlp.md) — AgentKeyAuthGuard (companion guard for proxy)
