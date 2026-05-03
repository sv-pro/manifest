# backend/health/

Minimal health check endpoint.

## Files

- `health.controller.ts` — `GET /api/v1/health` decorated with `@Public()`, always returns 200 with `{ status: 'ok' }`
- `health.module.ts`

Used by load balancers, Docker health checks, and Railway's health probe.

## See Also

- [../../api.md](../../api.md) — full API reference
