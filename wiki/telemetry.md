# Telemetry

Self-hosted Manifest installs send one anonymous aggregate usage report per 24 hours. Cloud/dev instances never send telemetry.

## When It Fires

- Only when `NODE_ENV=production`
- Only when `MANIFEST_TELEMETRY_DISABLED` is not set
- At most once per 24 hours (checked hourly, skipped if last send < 24h ago)
- Jitter on first send to avoid thundering-herd from many instances restarting

The `@Cron(CronExpression.EVERY_HOUR)` tick + timestamp check pattern survives restarts without missing windows.

## Payload (schema v1)

```json
{
  "schema_version": 1,
  "install_id": "<random UUIDv4, persisted in install_metadata>",
  "manifest_version": "6.0.1",

  "messages_total": 1234,
  "messages_by_provider": { "anthropic": 800, "openai": 400, "custom": 34 },
  "messages_by_tier": { "simple": 600, "standard": 400, "complex": 200, "reasoning": 34 },
  "messages_by_auth_type": { "api_key": 1000, "subscription": 234 },
  "tokens_input_total": 2500000,
  "tokens_output_total": 800000,

  "agents_total": 5,
  "agents_by_platform": { "openclaw": 3, "other": 2 },

  "platform": "linux",
  "arch": "x64"
}
```

### Bucketing

- Unknown provider values → `"custom"`
- NULL provider → `"unknown"`
- NULL tier → `"unknown"`
- Provider attribution uses `PROVIDER_BY_ID_OR_ALIAS` (same map as routing)

## Explicitly Never Sent

- Tenant or user IDs
- Email addresses
- API keys or OAuth tokens
- Message contents or prompts
- Model names
- Custom provider URLs
- Raw IP addresses
- Agent names

## Opt-Out

Set `MANIFEST_TELEMETRY_DISABLED=1` in `.env`. Also automatically disabled when `NODE_ENV !== 'production'`.

## Install ID

`InstallIdService` generates a random UUIDv4 on first startup and persists it in the `install_metadata` table. The ID is stable across restarts but has no connection to any user or tenant data.

## Extending the Payload

Bump `TELEMETRY_SCHEMA_VERSION` in `telemetry/telemetry.config.ts` and add fields additively. The telemetry ingest endpoint rejects unknown `schema_version` values with 400, keeping downgrades safe.

## Module Location

`packages/backend/src/telemetry/` — contains `telemetry.service.ts`, `install-id.service.ts`, `payload-builder.service.ts`, `telemetry.config.ts`, and a `dto/` for type safety.

## See Also

- [deployment.md](deployment.md) — self-hosted vs. cloud mode
- [entities.md](entities.md) — InstallMetadata entity
- [code/backend/telemetry.md](code/backend/telemetry.md) — module internals
