# Multi-Tenancy

Manifest is multi-tenant: each user owns a Tenant, and agents + routing config are scoped to that tenant.

## Ownership Hierarchy

```
User (Better Auth — user table)
  │
  └── Tenant (tenants table)
        │  tenant.name = user.id
        │
        └── Agent (agents table)
              │  unique on [tenant_id, name]
              │
              ├── AgentApiKey (agent_api_keys) — mnfst_* token for proxy auth
              ├── UserProvider (user_providers) — connected LLM providers + cached models
              ├── TierAssignment (tier_assignments) — tier → model/provider mapping
              ├── SpecificityAssignment (specificity_assignments) — category config
              ├── HeaderTier (header_tiers) — request-header-based routing rules
              └── AgentMessage (agent_messages) — telemetry (requests, tokens, costs)
```

## Tenant Creation

Tenants are created **lazily** on first agent creation via `ApiKeyGeneratorService.onboardAgent()`. The flow is atomic (single DB transaction):

1. Check if tenant exists for `user.id`
2. If not → create `Tenant { name: user.id }`
3. Create `Agent { tenantId, name }`
4. Create `AgentApiKey { agentId, keyHash }`
5. Return the plaintext key (shown once; never stored)

## Data Isolation

All analytics and routing queries filter by user via `addTenantFilter(qb, userId)` from `packages/backend/src/analytics/services/query-helpers.ts`. The `userId` comes from the `@CurrentUser()` decorator on each controller method — no endpoint can cross tenant boundaries.

```typescript
// Pattern used throughout analytics services:
const qb = this.repo.createQueryBuilder('m');
addTenantFilter(qb, userId);   // always applied before any other WHERE
```

## Agent Duplication

`POST /api/v1/agents/:name/duplicate` copies an agent's full routing config (providers, tier assignments, specificity assignments, custom providers, header tiers) to a new agent owned by the same tenant. Preview available via `GET /api/v1/agents/:name/duplicate-preview`.

## Agent Deletion

Agents are soft-deleted (deleted_at column). The agent's API key, provider configs, tier assignments, and messages are cascade-deleted. The tenant is not deleted when the last agent is removed.

## See Also

- [authentication.md](authentication.md) — how users and agents authenticate
- [entities.md](entities.md) — TypeORM entity details
- [data-flow.md](data-flow.md) — onboarding flow
- [code/backend/analytics.md](code/backend/analytics.md) — query helpers
