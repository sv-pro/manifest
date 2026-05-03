# backend/auth/

Better Auth integration: session management, social OAuth, and the `@CurrentUser()` decorator.

## Files

| File | Description |
|------|-------------|
| `auth.instance.ts` | Creates and exports the Better Auth singleton. Reads `process.env` at import time — must be available before module load. |
| `auth.module.ts` | NestJS module that registers `SessionGuard` as `APP_GUARD` (global). |
| `session.guard.ts` | Validates Better Auth cookie sessions via `auth.api.getSession()`. Attaches `request.user` and `request.session`. |
| `current-user.decorator.ts` | `@CurrentUser()` parameter decorator that reads `request.user` attached by `SessionGuard`. |

## auth.instance.ts

```typescript
const auth = betterAuth({
  database: pool,           // pg.Pool shared with TypeORM
  emailAndPassword: { enabled: true },
  socialProviders: {
    google:  { … },        // only if GOOGLE_CLIENT_ID + GOOGLE_CLIENT_SECRET set
    github:  { … },        // only if GITHUB_CLIENT_ID + GITHUB_CLIENT_SECRET set
    discord: { … },        // only if DISCORD_CLIENT_ID + DISCORD_CLIENT_SECRET set
  },
  trustedOrigins: [CORS_ORIGIN, BETTER_AUTH_URL, …],
  emailVerification: { sendVerificationEmail: … },  // uses Mailgun if configured
});

export type AuthSession = typeof auth.$Infer.Session;
export type AuthUser    = typeof auth.$Infer.Session.user;
```

Better Auth manages its own tables (`user`, `session`, `account`, `verification`) via `ctx.runMigrations()` — independent of TypeORM migrations.

## SessionGuard

Priority order in `canActivate()`:

1. Is route `@Public()`? → skip (return true)
2. Is session cookie present and valid? → attach `request.user`, return true
3. No session → return false (falls through to `ApiKeyGuard`)

## Gotchas

- `auth.instance.ts` is imported at module initialization time, which means `process.env` must already contain `BETTER_AUTH_SECRET` **before** the first `import`. Use `NODE_OPTIONS='-r dotenv/config'` in dev.
- Social login only works on the port that `BETTER_AUTH_URL` points to (`:3001` by default). Vite's `:3000` proxy does not receive the OAuth callback.

## See Also

- [../../authentication.md](../../authentication.md) — full auth concept page
- [common.md](common.md) — ApiKeyGuard
- [otlp.md](otlp.md) — agent API key guard
