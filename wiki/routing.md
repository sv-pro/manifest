# LLM Routing

Manifest uses a two-layer routing system to pick the cheapest model that can handle each request.

## Layer 1: Complexity Tiers

Always active. Every request is scored and assigned one of four tiers:

| Tier | Intent |
|------|--------|
| `simple` | Factual lookups, short transforms, trivial completions |
| `standard` | Typical multi-step tasks, moderate reasoning |
| `complex` | Multi-document synthesis, hard code generation, long outputs |
| `reasoning` | Chain-of-thought problems, formal logic, deep analysis |

Each agent has a **TierAssignment** for every tier slot, mapping `tier → provider + model`. These are auto-assigned by `TierAutoAssignService` when providers connect, and can be manually overridden.

A fifth slot, `default`, is used as a fallback when no tier-specific assignment exists.

See [scoring.md](scoring.md) for how the tier is determined.

## Layer 2: Specificity Routing (opt-in)

When enabled for a category, specificity **overrides** the complexity tier for matching requests. Each of the 9 categories gets its own model assignment:

`coding` · `web_browsing` · `data_analysis` · `image_generation` · `video_generation` · `social_media` · `email_management` · `calendar_management` · `trading`

See [specificity.md](specificity.md) for detection logic.

## Resolution Order

```
Incoming request
  ├── Any specificity category active AND request matches category?
  │     YES → use SpecificityAssignment model
  │     NO  ↓
  └── Complexity tier from scorer
        └── Look up TierAssignment for tier
              └── Provider + model → forward
```

## Fallback Chains

Each tier assignment can have an ordered list of fallback models. If the primary fails with a retryable error (429, 5xx):

```
Primary model fails
  └── Try fallback[0]
        └── Try fallback[1]
              └── … → 503 if all exhausted
```

The `fallback_from_model` column in `agent_messages` records which model was actually used.

## Header-Based Tier Routing

`HeaderTier` rules let callers force a specific tier via request headers. Rules are evaluated in priority order before the scoring step. Useful for agents that know the complexity of a request better than the scorer does.

`GET/POST/PUT/PATCH/DELETE /api/v1/routing/:agentName/header-tiers`

## Custom Providers

Users can register arbitrary OpenAI-compatible endpoints as custom providers (`custom:<uuid>` path). The probe endpoint (`POST .../custom-providers/probe`) tests connectivity before saving.

## Provider Key Management

Each `UserProvider` row stores the encrypted API key (AES-256-GCM, key derived from `BETTER_AUTH_SECRET`). `ProviderKeyService` handles encrypt/decrypt. Keys are never logged or returned in API responses after initial creation.

## Routing Cache

`RoutingCacheService` caches the resolved `provider + model` for a given `(agentId, tier)` tuple. Cache is invalidated by `RoutingInvalidationService` when tier assignments or provider configs change.

## Session Momentum

`SessionMomentumService` applies tier stickiness: if recent turns in the same conversation scored `complex`, the current turn's tier is biased upward even if the current message scores lower. This prevents jarring mid-conversation model switches.

## See Also

- [scoring.md](scoring.md) — complexity dimension engine
- [specificity.md](specificity.md) — category detection
- [providers.md](providers.md) — provider registry
- [models.md](models.md) — model discovery
- [data-flow.md](data-flow.md) — routing in the full request lifecycle
- [code/backend/routing.md](code/backend/routing.md) — routing module internals
