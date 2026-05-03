# Entities

All 23 TypeORM entities in `packages/backend/src/entities/`. Every schema change goes through a migration — `synchronize` is permanently disabled.

## Core Ownership

| Entity | Table | Description |
|--------|-------|-------------|
| `Tenant` | `tenants` | Multi-tenant root; `name = user.id`. Created on first agent. |
| `Agent` | `agents` | AI agent owned by a tenant. Unique on `[tenant_id, name]`. Soft-deleted (`deleted_at`). |
| `AgentApiKey` | `agent_api_keys` | One-to-one with Agent. Stores hashed `mnfst_*` key (scrypt). |
| `ApiKey` | `api_keys` | Dashboard API keys (X-API-Key header auth). |

## Telemetry

| Entity | Table | Description |
|--------|-------|-------------|
| `AgentMessage` | `agent_messages` | Primary telemetry row per proxied request. Key columns: `model`, `routing_tier`, `routing_reason`, `specificity_category`, `auth_type`, `fallback_from_model`, `input_tokens`, `output_tokens`, `cost_usd`, `duration_ms`. |
| `LlmCall` | `llm_calls` | Individual LLM call within a message (for multi-turn or retry scenarios). |
| `ToolExecution` | `tool_executions` | Tool/function call records linked to a message. |
| `AgentLog` | `agent_logs` | Free-form log entries for an agent. |

## Routing Configuration

| Entity | Table | Description |
|--------|-------|-------------|
| `UserProvider` | `user_providers` | Connected LLM provider for an agent. Stores encrypted API key, provider ID, `cached_models` JSONB. |
| `TierAssignment` | `tier_assignments` | Maps `(agent, tier)` → `(provider, model)`. Also holds `fallback_models` array. 5 slots per agent (simple/standard/complex/reasoning/default). |
| `SpecificityAssignment` | `specificity_assignments` | Per-category routing config for an agent. Enabled flag + model assignment. |
| `HeaderTier` | `header_tiers` | Header-based tier routing rule. `header_name + header_value → tier`. Ordered by priority. |
| `CustomProvider` | `custom_providers` | User-defined OpenAI-compatible endpoint. `baseUrl`, `apiKey` (encrypted), `modelId`. |

## Notifications

| Entity | Table | Description |
|--------|-------|-------------|
| `NotificationRule` | `notification_rules` | Alert threshold rule (e.g. cost > $5/day). Stores metric type, threshold, window, enabled flag. |
| `NotificationLog` | `notification_logs` | Delivery history for a fired notification. Status, timestamps, error info. |
| `EmailProviderConfig` | `email_provider_configs` | Email provider credentials. Supports Mailgun, Resend, SMTP. Keys encrypted. |

## Telemetry Infrastructure

| Entity | Table | Description |
|--------|-------|-------------|
| `InstallMetadata` | `install_metadata` | Single row per installation. Stores `install_id` (random UUIDv4) for anonymous telemetry. |

## Key Columns in `agent_messages`

This is the most queried table. Critical columns for the analytics layer:

```
id, agent_id, tenant_id
model                    — raw model name from provider response
routing_tier             — simple | standard | complex | reasoning
routing_reason           — scored | specificity | heartbeat | header | fallback
specificity_category     — coding | trading | … | null
auth_type                — api_key | subscription | local
fallback_from_model      — original model if a fallback was used
input_tokens, output_tokens, cache_read_tokens
cost_usd                 — computed from pricing cache at ingest time
duration_ms              — end-to-end latency
request_headers          — JSONB of sanitized headers
created_at
```

## Message Row Projection Contract

Any endpoint returning rows rendered by `MessageTable` / `ModelCell` **must** use `selectMessageRowColumns()` from `query-helpers.ts`. This helper is the single source of truth for the shared column set. See [architecture.md](architecture.md) and [development.md](development.md#shared-projection).

## See Also

- [multi-tenancy.md](multi-tenancy.md) — ownership hierarchy
- [data-flow.md](data-flow.md) — how messages are written
- [development.md](development.md) — migration workflow
- [code/backend/entities.md](code/backend/entities.md) — entity module details
