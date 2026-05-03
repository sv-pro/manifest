# backend/database/

TypeORM configuration, migration runner, database seeder, and background sync services.

## Files

| File | Description |
|------|-------------|
| `database.module.ts` | Registers TypeORM with PostgreSQL, imports all 23 entities and 100+ migrations. `synchronize: false`, `migrationsRun: true`. |
| `datasource.ts` | CLI `DataSource` instance used by `migration:generate` / `migration:run` commands. |
| `database-seeder.service.ts` | Seeds demo data on startup when `SEED_DATA=true`. Idempotent. |
| `seed-messages.ts` | Static sample telemetry messages for the demo agent. |
| `pricing-sync.service.ts` | Fetches and caches OpenRouter pricing data. Runs on startup + daily `@Cron`. |
| `ollama-sync.service.ts` | Syncs the Ollama local model list from `OLLAMA_HOST`. |
| `quality-score.util.ts` | Computes a numeric quality score for a model name string (used for tier auto-assignment). |

## Migration Workflow

Migrations live in `src/database/migrations/`. Each file is a TypeORM `MigrationInterface` with a timestamp prefix.

```bash
# Generate after entity change:
npm run migration:generate -- src/database/migrations/DescriptiveName

# Must also: import in database.module.ts + add to migrations array
```

See [../../development.md](../../development.md#database-migrations) for the full workflow.

## Pricing Sync

`PricingSyncService` fetches `https://openrouter.ai/api/v1/models` (no key required) and stores the result in-memory. All model pricing in Manifest derives from this cache — there is no hardcoded pricing data. Cache is invalidated by `POST /api/v1/routing/pricing/refresh`.

## Seeded Data

| Resource | Value |
|----------|-------|
| Admin | `admin@manifest.build` / `manifest` |
| Tenant | `seed-tenant-001` |
| Agent | `demo-agent` with key `dev-otlp-key-001` |
| API Key | `dev-api-key-manifest-001` |
| Messages | ~50 sample telemetry messages |

## See Also

- [entities.md](entities.md) — all 23 entities
- [../../development.md](../../development.md) — migration commands
- [../../models.md](../../models.md) — pricing cache usage
