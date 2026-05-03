# backend/public-stats/

Unauthenticated aggregate statistics for the Manifest marketing site and public dashboards.

## Files

- `public-stats.controller.ts` — four `@Public()` endpoints
- `public-stats.service.ts` — queries + caches aggregate stats
- `public-stats.module.ts`

## Endpoints

| Route | Description |
|-------|-------------|
| `GET /api/v1/public/usage` | Total message count + top models (cached with `PUBLIC_STATS_CACHE_TTL_MS`) |
| `GET /api/v1/public/free-models` | Free model list (unauthenticated version) |
| `GET /api/v1/public/provider-tokens` | Daily token counts by provider |
| `GET /api/v1/public/free-providers` | Providers with free tier models |

All endpoints apply aggressive caching (`CacheInterceptor`) to avoid DB load from unauthenticated traffic.

## See Also

- [free-models.md](free-models.md) — authenticated free models
- [../../api.md](../../api.md) — full endpoint reference
