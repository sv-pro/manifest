# backend/otlp/

Agent API key authentication guard and first-use onboarding service.

## Files

| File | Description |
|------|-------------|
| `guards/agent-key-auth.guard.ts` | Validates `Bearer mnfst_*` tokens on proxy endpoints. 5-min in-memory cache. |
| `services/api-key.service.ts` | `ApiKeyGeneratorService.onboardAgent()` — creates tenant + agent + key in one transaction. |
| `otlp-deprecated.controller.ts` | Handles legacy `POST /otlp/chat/completions` path (redirects to proxy). |

## AgentKeyAuthGuard

Applied to the proxy routes (`/v1/chat/completions`, `/v1/responses`, `/api/v1/routing/resolve`).

```
Bearer token received
  ├── In 5-min cache? → return cached agent
  ├── Lookup agent_api_keys by key hash
  ├── Not found AND selfhosted mode AND loopback IP? → auto-onboard
  ├── Not found otherwise → 401
  └── Found → cache + attach to request
```

## Onboarding Flow

`onboardAgent(userId, agentName)` — atomic transaction:

1. Find or create `Tenant { name: userId }`
2. Create `Agent { tenantId, name: agentName }`
3. Generate random key, hash it with scrypt
4. Create `AgentApiKey { agentId, keyHash }`
5. Return plaintext key (shown once)

## See Also

- [../../authentication.md](../../authentication.md) — full guard chain
- [../../multi-tenancy.md](../../multi-tenancy.md) — onboarding in context
- [routing.md](routing.md) — proxy module that uses the guard
