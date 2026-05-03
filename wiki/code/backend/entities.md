# backend/entities/

All 23 TypeORM entity files. Each maps to a PostgreSQL table. See [../../entities.md](../../entities.md) for semantic descriptions and key columns.

## File List

```
agent-api-key.entity.ts        — agent_api_keys: mnfst_* hashed keys
agent-log.entity.ts            — agent_logs: free-form log entries
agent-message.entity.ts        — agent_messages: primary telemetry row
agent.entity.ts                — agents: AI agent, belongs to tenant
api-key.entity.ts              — api_keys: dashboard X-API-Key records
custom-provider.entity.ts      — custom_providers: user-defined OpenAI-compatible endpoints
email-provider-config.entity.ts— email_provider_configs: Mailgun/Resend/SMTP credentials
header-tier.entity.ts          — header_tiers: request-header routing rules
install-metadata.entity.ts     — install_metadata: single-row install ID for telemetry
llm-call.entity.ts             — llm_calls: individual LLM calls within a message
notification-log.entity.ts     — notification_logs: delivery history
notification-rule.entity.ts    — notification_rules: alert threshold configs
specificity-assignment.entity.ts — specificity_assignments: per-category routing config
tenant.entity.ts               — tenants: multi-tenant root, name=user.id
tier-assignment.entity.ts      — tier_assignments: tier → model mapping
tool-execution.entity.ts       — tool_executions: tool/function call records
user-provider.entity.ts        — user_providers: connected provider + encrypted key + cached_models
```

## Conventions

- All PKs are `uuid` generated at DB level
- Timestamps: `created_at` on all tables; `updated_at` on mutable tables
- Soft deletes: `deleted_at` on `agents`
- Encrypted columns: `user_providers.api_key`, `custom_providers.api_key`, `email_provider_configs.api_key` — AES-256-GCM via `CryptoUtil`
- JSONB columns: `user_providers.cached_models`, `tier_assignments.fallback_models`, `header_tiers.*`

## See Also

- [../../entities.md](../../entities.md) — role and key column descriptions
- [database.md](database.md) — migration workflow
- [../../multi-tenancy.md](../../multi-tenancy.md) — ownership hierarchy
