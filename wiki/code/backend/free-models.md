# backend/free-models/

Discovers and serves a list of free (zero-cost) LLM models.

## Files

| File | Description |
|------|-------------|
| `free-models.service.ts` | Aggregates free models across all connected providers for the current user |
| `free-models.controller.ts` | `GET /api/v1/free-models` — authenticated; returns free models for the session user |
| `free-models-sync.service.ts` | Background sync: periodically refreshes the free model list from provider APIs |
| `free-models-provider-metadata.ts` | Static metadata about which providers offer free tiers |
| `free-models.module.ts` | NestJS module wiring |

## Free Model Detection

A model is "free" if its `input_cost` and `output_cost` are both zero (or null) in the OpenRouter pricing cache. The service also checks provider-level free tier metadata.

The **public** free model list (unauthenticated) is served by `public-stats/public-stats.controller.ts` at `GET /api/v1/public/free-models`.

## See Also

- [public-stats.md](public-stats.md) — public version of free models
- [../../models.md](../../models.md) — model pricing data source
