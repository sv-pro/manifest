# backend/telemetry/

Anonymous aggregate usage telemetry for self-hosted installs.

## Files

| File | Description |
|------|-------------|
| `telemetry.service.ts` | Cron driver: checks 24h window, calls payload builder, POSTs to endpoint |
| `install-id.service.ts` | Generates and persists a random UUIDv4 in `install_metadata` |
| `payload-builder.service.ts` | Queries `agent_messages` for aggregates, builds the payload object |
| `telemetry.config.ts` | `TELEMETRY_SCHEMA_VERSION` constant, endpoint URL, opt-out check |
| `telemetry.module.ts` | NestJS module wiring |
| `dto/` | TypeScript types for the payload |

## See Also

- [../../telemetry.md](../../telemetry.md) — full concept page (payload fields, opt-out, cadence)
- [../entities.md](entities.md) — InstallMetadata entity
