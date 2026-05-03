# backend/main.ts + app.module.ts

Entry point and root module of the NestJS application.

## main.ts — Bootstrap

Key setup order (order matters):

1. **Disable body parsing** — `bodyParser: false` so Better Auth gets raw body
2. **Helmet** — strict security headers including CSP (`'self'` only, no CDN)
3. **Compression** — gzip for all responses
4. **CORS** — enabled only in `NODE_ENV=development` pointing at `CORS_ORIGIN`
5. **Auth rate limiting** — express-rate-limit on `/api/auth/sign-in` and `/api/auth/sign-up` (separate stricter limits)
6. **Better Auth middleware** — mounted at `/api/auth/*splat` before `express.json()`
7. **express.json() + express.urlencoded()** — re-added after Better Auth
8. **ValidationPipe** — global, `whitelist: true`, `forbidNonWhitelisted: true`
9. **SPA fallback filter** — serves `index.html` for non-API routes in production
10. **Listen** — on `PORT` / `BIND_ADDRESS` from env

## app.module.ts — Root Module

Imports every feature module. Notable configuration:

- **CacheModule** — in-memory cache used by `RoutingCacheService`, `AgentCacheInterceptor`, `UserCacheInterceptor`
- **ThrottlerModule** — global rate limiting (`THROTTLE_TTL` / `THROTTLE_LIMIT`)
- **ServeStaticModule** — serves `packages/frontend/dist/` in production with SPA fallback
- **Global guards** (applied in order): `SessionGuard`, `ApiKeyGuard`, `ThrottlerGuard`
- **ScheduleModule** — enables `@Cron` decorators for pricing sync, notification check, telemetry

## See Also

- [../../architecture.md](../../architecture.md) — body parsing, CORS, CSP
- [auth.md](auth.md) — SessionGuard, Better Auth
- [../../authentication.md](../../authentication.md) — guard chain
