# backend/routing/

LLM proxy, tier/specificity configuration, provider management, and model resolution. The largest module in the backend.

## Subdirectories

| Path | Description |
|------|-------------|
| `proxy/` | OpenAI-compatible HTTP proxy — the core traffic path |
| `routing-core/` | Tier/provider/specificity services, routing cache, invalidation |
| `resolve/` | Scoring-based model resolution endpoint |
| `custom-provider/` | User-defined custom LLM provider CRUD |
| `header-tiers/` | Request-header-based tier routing rules |
| `oauth/` | OpenAI OAuth and MiniMax device-code OAuth flows |

## Top-Level Controllers

| Controller | Endpoints |
|-----------|-----------|
| `provider.controller.ts` | Connect/disconnect providers, status, deactivate-all |
| `tier.controller.ts` | Tier overrides, fallbacks, complexity toggle |
| `specificity.controller.ts` | Specificity category assignments, toggle, reset |
| `model.controller.ts` | Available models, refresh models, Ollama sync, pricing health |
| `copilot.controller.ts` | GitHub Copilot device-code + token-poll |

## proxy/

The critical path for all proxied LLM requests.

| File | Description |
|------|-------------|
| `proxy.controller.ts` | `POST /v1/chat/completions` and `POST /v1/responses`. Rate-limit, caller classification, streaming/non-streaming dispatch. |
| `proxy.service.ts` | Orchestrates scoring → specificity → resolution → key decrypt → forward → record. |
| `proxy-fallback.service.ts` | Tries fallback chain when primary model fails. |
| `proxy-message-recorder.ts` | Persists `AgentMessage`, `LlmCall`, `ToolExecution`. Emits SSE event. |
| `proxy-response-handler.ts` | Handles streaming SSE pipe-through and non-streaming JSON assembly. |
| `provider-client.ts` | HTTP client that forwards to provider. |
| `anthropic-adapter.ts` | Converts OpenAI ↔ Anthropic format (requests and responses). |
| `chatgpt-adapter.ts` | OpenAI protocol utilities and normalization. |
| `google-adapter.ts` | Converts OpenAI ↔ Google Gemini format. |
| `caller-classifier.ts` | Identifies browser vs. programmatic callers for error formatting. |
| `session-momentum.service.ts` | Tier stickiness across turns in the same conversation. |
| `stream-writer.ts` | Writes SSE chunks to the response stream. |
| `proxy-message-dedup.ts` | Detects and drops duplicate requests (idempotency). |
| `proxy-rate-limiter.ts` | Per-agent per-minute rate limiting for the proxy. |
| `copilot-token.service.ts` | Manages GitHub Copilot OAuth token lifecycle. |
| `provider-endpoints.ts` | Base URL registry for each provider. |

## routing-core/

| File | Description |
|------|-------------|
| `provider.service.ts` | UserProvider CRUD: connect, disconnect, encrypt key, trigger model discovery. |
| `provider-key.service.ts` | AES-256-GCM encrypt/decrypt for stored provider API keys. |
| `tier.service.ts` | TierAssignment CRUD: ensure all 5 slots exist, apply overrides, manage fallbacks. |
| `tier-auto-assign.service.ts` | Auto-assigns models to tiers based on quality score when provider connects. |
| `specificity.service.ts` | SpecificityAssignment CRUD: enable/disable categories, set model overrides. |
| `specificity-penalty.service.ts` | Applies penalty to weak specificity signals during detection. |
| `resolve-agent.service.ts` | Resolves agent entity from `(userId, agentName)`, used by all routing controllers. |
| `routing-cache.service.ts` | Caches `(agentId, tier) → (provider, model)` resolution. |
| `routing-invalidation.service.ts` | Clears routing cache when tier/provider config changes. |
| `route-helpers.ts` | Shared helpers: `buildModelRoute()`, `providerForModel()`. |

## See Also

- [../../routing.md](../../routing.md) — routing concept page
- [../../scoring.md](../../scoring.md) — scoring feeds into proxy
- [../../providers.md](../../providers.md) — provider registry
- [../../data-flow.md](../../data-flow.md) — full request lifecycle
