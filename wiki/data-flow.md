# Data Flow

End-to-end lifecycle of a proxied LLM request, from agent to provider and back.

## Happy Path

```
Agent (OpenClaw / OpenAI SDK / curl / …)
  │
  │  POST /v1/chat/completions
  │  Authorization: Bearer mnfst_<key>
  ▼
AgentKeyAuthGuard
  ├── Looks up key hash in agent_api_keys (5-min in-memory cache)
  ├── Attaches agent + tenant to request
  └── 401 if key invalid or not found
  ▼
ProxyController.chatCompletions()
  ├── Extract caller type (CallerClassifier: browser vs. tool)
  ├── Rate-limit check (per-agent, per-minute)
  └── Delegates to ProxyService.forward()
  ▼
ProxyService.forward()
  ├── 1. scoreRequest()          → complexity tier (simple/standard/complex/reasoning)
  ├── 2. specDetector.detect()   → specificity category (coding/trading/… or null)
  ├── 3. SessionMomentum         → apply tier stickiness across turns
  ├── 4. ResolutionService       → resolve final provider + model from tier/specificity config
  ├── 5. ProviderKeyService      → decrypt AES-256-GCM API key for chosen provider
  └── 6. ProviderClient.forward()
        ├── Adapter: convert OpenAI payload → provider format (Anthropic/Google/ChatGPT/…)
        ├── HTTP request to provider
        └── Adapter: convert provider response → OpenAI format
  ▼
ProxyMessageRecorder
  ├── Persist AgentMessage row (model, tier, tokens, cost, latency, headers)
  ├── Persist LlmCall + ToolExecution rows if present
  └── Emit SSE event for real-time dashboard update
  ▼
ProxyResponseHandler
  ├── Streaming: pipe SSE chunks back to agent
  └── Non-streaming: return JSON response
```

## Fallback Path

If the primary provider/model returns a 429, 500, 502, or 503:

```
ProxyFallbackService
  ├── Read fallback chain from tier_assignments.fallback_models
  ├── Try each fallback in order
  ├── Record which model was actually used (fallback_from_model column)
  └── If all fallbacks exhausted → 503 with friendly message
```

## Specificity vs. Complexity Resolution

```
Has active specificity category for this agent?
  ├── YES → use specificity assignment's model (skip complexity scoring)
  └── NO  → use complexity tier assignment
               ├── simple tier model
               ├── standard tier model
               ├── complex tier model
               └── reasoning tier model
```

## Ingest / Onboarding Flow (first use)

```
Agent sends first request with unknown Bearer token
  ▼
AgentKeyAuthGuard
  ├── Key not found in DB
  └── ApiKeyGeneratorService.onboardAgent()
        ├── Create Tenant (if user is new)
        ├── Create Agent
        ├── Create AgentApiKey (mnfst_*)
        └── Return agent context
```

## Analytics Write Path

Every `AgentMessage` row drives the dashboard:

```
agent_messages table
  ├── Overview: aggregated token/cost/message counts
  ├── Tokens page: timeseries by tier/provider
  ├── Costs page: timeseries + savings vs baseline
  ├── Messages page: paginated log with routing details
  └── Notifications: cron checks threshold rules every hour
```

## See Also

- [routing.md](routing.md) — tier + specificity resolution detail
- [scoring.md](scoring.md) — complexity dimension engine
- [specificity.md](specificity.md) — category detection
- [providers.md](providers.md) — provider adapters
- [authentication.md](authentication.md) — guard chain
- [code/backend/routing.md](code/backend/routing.md) — proxy module internals
