# backend/setup/

First-run setup flow for creating the initial admin account.

## Files

- `setup.controller.ts` — two `@Public()` endpoints
- `setup.service.ts` — checks if admin exists; creates admin via Better Auth
- `setup.module.ts`
- `dto/` — input validation for admin creation

## Endpoints

| Route | Description |
|-------|-------------|
| `GET /api/v1/setup/status` | Returns `{ configured: boolean }` — is admin already created? |
| `POST /api/v1/setup/admin` | Create the initial admin user (disabled after first successful call) |

The frontend shows a setup wizard on first load if `status.configured === false`.

## See Also

- [../../development.md](../../development.md) — SEED_DATA alternative for dev
- [auth.md](auth.md) — Better Auth user creation
