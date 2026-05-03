# manifest-shared

Canonical TypeScript types and constants consumed by both the backend and the frontend. The only package in the monorepo that is `private: true` but has no runtime dependencies other than TypeScript itself.

## Purpose

Prevents backend and frontend from drifting on any shared fact. If a constant, type, or business rule is needed by both sides, it lives here and is imported from `manifest-shared`.

## Modules

| Module | Page | Description |
|--------|------|-------------|
| `providers.ts` | [providers.md](providers.md) | `SHARED_PROVIDERS` — canonical provider registry (single source of truth) |
| `specificity.ts` | [specificity.md](specificity.md) | `SPECIFICITY_CATEGORIES` — the 9 task-type category IDs |
| `agent-type.ts` | [agent-type.md](agent-type.md) | Agent platforms, categories, labels, icons |
| `tiers.ts` | [tiers.md](tiers.md) | Complexity tier names and descriptions |
| `tier-colors.ts` | [tier-colors.md](tier-colors.md) | UI color constants per tier |
| `subscription/` | [subscription.md](subscription.md) | Subscription provider configs, types, and helpers |
| `provider-inference.ts` | — | Infer provider from model name string |
| `resolve-response.ts` | — | TypeScript types for the `/routing/resolve` response |
| `model-route.ts` | — | `ModelRoute` type used across routing services |
| `api-key.ts` | — | API key format constants |
| `auth-types.ts` | — | `AuthType` enum (`api_key` / `subscription` / `local`) |

## Key Exports

```typescript
// Most important exports consumed downstream:
export { SHARED_PROVIDERS, SHARED_PROVIDER_BY_ID, SHARED_PROVIDER_BY_ID_OR_ALIAS }
export { SPECIFICITY_CATEGORIES, type SpecificityCategory }
export { AGENT_PLATFORMS, AGENT_CATEGORIES, PLATFORM_LABELS, PLATFORM_ICONS }
export { LOCAL_SERVER_HINTS }
export { normalizeProviderName }
```

## See Also

- [../backend/index.md](../backend/index.md) — how backend consumes shared
- [../frontend/index.md](../frontend/index.md) — how frontend consumes shared
- [../../providers.md](../../providers.md) — provider registry concept page
