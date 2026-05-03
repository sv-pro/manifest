# backend/model-discovery/

Fetches, enriches, and caches model lists from provider APIs.

## Files

| File | Description |
|------|-------------|
| `model-discovery.service.ts` | Orchestrator: decrypt key → fetch → enrich → cache → trigger tier recalc |
| `model-discovery.module.ts` | NestJS module wiring |
| `provider-model-fetcher.service.ts` | Config-driven fetcher with one `FetcherConfig` per provider |
| `model-fallback.ts` | Builds model list from OpenRouter cache when native /models fails |
| `model-fetcher.ts` | Low-level HTTP fetcher with retry logic |
| `known-model-prices.ts` | Hardcoded price hints for a few models not in OpenRouter (last resort) |
| `filter-non-chat-models.ts` | Filters embedding, image, audio models from chat-only lists |
| `anthropic-subscription-probe.ts` | Detects Claude.ai subscription key vs. paid API key |
| `opencode-go-catalog.service.ts` | Fetches OpenCode Go's model catalog |
| `provider-model-registry.service.ts` | In-memory `(userId, providerId) → ModelList` cache |

## ProviderModelFetcherService

Has a `FetcherConfig` for each provider defining:
- `endpoint`: URL template
- `parser`: format-specific response parser (`openai-compat`, `anthropic`, `gemini`, `openrouter`, `ollama`, `opencode-go`)
- `requiresKey`: whether to pass the API key

When a provider returns 0 models (no `/models` endpoint), `model-fallback.ts` builds a list from the OpenRouter pricing cache filtered by the provider's `openRouterPrefixes`.

## ModelDiscoveryService

Called by `ProviderService.connect()` synchronously. Returns enriched model list immediately so the UI shows models right after connection.

```
fetch(providerId, encryptedKey)
  ├── decrypt key
  ├── ProviderModelFetcherService.fetch()
  ├── For each model: enrich with pricing + quality score
  ├── Store in user_providers.cached_models (JSONB)
  └── TierAutoAssignService.recalculate()
```

## AnthropicSubscriptionProbeService

Issues a minimal API call to detect if the key provides Claude.ai subscription capabilities (higher rate limits, different model access). Result stored in `user_providers.subscription_type`.

## See Also

- [../../models.md](../../models.md) — model discovery concept page
- [../../providers.md](../../providers.md) — provider registry
- [database.md](database.md) — PricingSyncService
