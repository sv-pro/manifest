# Models

Model discovery, pricing, and tier assignment.

## Discovery Flow

```
User connects provider (POST /routing/:agent/providers)
  ▼
ModelDiscoveryService.discoverForAgent()
  ├── ProviderKeyService.decrypt(apiKey)
  ├── ProviderModelFetcherService.fetch(providerId, decryptedKey)
  │     ├── Calls provider's native /models endpoint
  │     │   (api.anthropic.com/v1/models, generativelanguage.googleapis.com/v1beta/models, …)
  │     └── If 0 models returned → buildFallbackModels() from OpenRouter pricing cache
  ├── Per model: ModelDiscoveryService.enrichModel()
  │     ├── Lookup pricing from PricingSyncService (OpenRouter cache)
  │     └── Compute quality score (quality-score.util.ts)
  └── Save to user_providers.cached_models (JSONB)
        └── TierAutoAssignService.recalculate() → auto-fill tier assignments
```

Discovery runs **synchronously** on provider connect so users see models immediately.

## Model Fetcher

`ProviderModelFetcherService` has a `FetcherConfig` per provider covering:
- Endpoint URL pattern
- Response format parser (OpenAI-compatible, Anthropic, Gemini, OpenRouter, Ollama, OpenCode Go)
- Whether the provider exposes a native `/models` endpoint

Providers without a native `/models` endpoint (e.g. MiniMax) fall back to the OpenRouter pricing cache.

## OpenRouter Pricing Cache

All pricing data comes from the **OpenRouter public API** (no key required). `PricingSyncService` fetches and caches it:
- On startup
- Daily via `@Cron`
- On-demand via `POST /api/v1/routing/pricing/refresh`

The cache is in-memory; no pricing data is hardcoded. `ModelPricingCacheService` attributes models to their real provider using `OPENROUTER_PREFIX_TO_PROVIDER`.

## Quality Score

`quality-score.util.ts` assigns a numeric quality score to each model, based on model name heuristics (size indicators, capability suffixes). Used by `TierAutoAssignService` to rank models for auto-assignment.

## Tier Auto-Assignment

When a provider connects (or models refresh), `TierAutoAssignService.recalculate()` automatically assigns models to tier slots:
- Sorts available models by quality score
- Maps quality buckets to tier slots (simple → cheapest, reasoning → highest quality)
- Preserves manual overrides (only fills empty slots)

## Where Models Appear

| Surface | Source | Details |
|---------|--------|---------|
| Model Prices page | `ModelPricingCacheService.getAll()` | All OpenRouter-known models, attributed to real providers |
| Routing → available models | `ModelDiscoveryService.getModelsForAgent()` | Only models from user's connected providers |
| Routing → tier assignments | `TierAutoAssignService.recalculate()` | Auto-assigned from discovered models |
| Messages / Overview | `agent_messages.model` column | Raw name from telemetry; display name from model-display cache |
| Free Models page | `FreeModelsService` | Models with zero cost across all connected providers |

## Anthropic Subscription Detection

`AnthropicSubscriptionProbeService` probes whether an Anthropic API key is a Claude.ai subscription key vs. a paid API key. Subscription keys have different rate limits and capabilities, which affects tier assignment and routing.

## Refresh Models

`POST /api/v1/routing/:agentName/refresh-models` re-fetches model lists from all connected providers for an agent, re-runs enrichment, and updates tier assignments.

## Provider Model Registry

`ProviderModelRegistryService` maintains an in-memory registry of model lists per `(userId, providerId)` tuple, avoiding redundant fetches within a session.

## See Also

- [providers.md](providers.md) — provider registry and connection
- [routing.md](routing.md) — tier assignments in routing
- [code/backend/model-discovery.md](code/backend/model-discovery.md) — module internals
