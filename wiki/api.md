# API Reference

Complete HTTP endpoint table. All `/api/v1/*` routes (except public ones) require either a Better Auth session cookie or `X-API-Key` header.

## Public Endpoints

| Method | Route | Description |
|--------|-------|-------------|
| GET | `/api/v1/health` | Health check (always 200) |
| ALL | `/api/auth/*` | Better Auth: login, register, OAuth, sessions, password reset |
| GET | `/api/v1/setup/status` | First-run setup: is admin account created? |
| POST | `/api/v1/setup/admin` | Create initial admin account (disabled after first use) |
| GET | `/api/v1/public/usage` | Aggregate usage stats (cached, no auth) |
| GET | `/api/v1/public/free-models` | Free model list |
| GET | `/api/v1/public/provider-tokens` | Provider daily token stats |
| GET | `/api/v1/public/free-providers` | Free provider list |
| GET | `/api/v1/github/stars` | GitHub star count |

## Analytics

| Method | Route | Description |
|--------|-------|-------------|
| GET | `/api/v1/overview` | Dashboard summary (messages, costs, tokens, recent activity) |
| GET | `/api/v1/tokens` | Token usage timeseries |
| GET | `/api/v1/costs` | Cost timeseries |
| GET | `/api/v1/savings` | Cost savings summary vs. baseline |
| GET | `/api/v1/savings/timeseries` | Savings over time |
| GET | `/api/v1/savings/baseline-candidates` | Models eligible as cost baseline |

## Messages

| Method | Route | Description |
|--------|-------|-------------|
| GET | `/api/v1/messages` | Paginated message log |
| GET | `/api/v1/messages/:id/details` | Full message detail: tokens, timing, headers, tool calls |
| PATCH | `/api/v1/messages/:id/feedback` | Submit routing feedback (good/bad tier) |
| DELETE | `/api/v1/messages/:id/feedback` | Clear routing feedback |
| PATCH | `/api/v1/messages/:id/miscategorized` | Flag specificity miscategorization |
| DELETE | `/api/v1/messages/:id/miscategorized` | Clear miscategorization flag |

## Agents

| Method | Route | Description |
|--------|-------|-------------|
| GET | `/api/v1/agents` | Agent list with sparklines |
| POST | `/api/v1/agents` | Create agent + generate API key |
| GET | `/api/v1/agents/:name` | Single agent detail |
| GET | `/api/v1/agents/:name/duplicate-preview` | Preview what a duplicate would look like |
| POST | `/api/v1/agents/:name/duplicate` | Duplicate agent with full routing config |
| DELETE | `/api/v1/agents/:name` | Soft-delete agent |
| GET | `/api/v1/agents/:name/key` | Retrieve API key (once after rotation) |
| POST | `/api/v1/agents/:name/rotate-key` | Rotate agent API key |
| PATCH | `/api/v1/agents/:name` | Rename agent |
| GET | `/api/v1/agent/:agentName/usage` | Per-agent token usage |
| GET | `/api/v1/agent/:agentName/costs` | Per-agent cost breakdown |

## Model Catalog

| Method | Route | Description |
|--------|-------|-------------|
| GET | `/api/v1/model-prices` | All models with pricing (from OpenRouter cache) |
| GET | `/api/v1/free-models` | Free/zero-cost models |

## Routing Configuration

| Method | Route | Description |
|--------|-------|-------------|
| GET | `/api/v1/routing/:agentName/status` | Provider connection status |
| GET/POST | `/api/v1/routing/:agentName/providers` | List / connect providers |
| DELETE | `/api/v1/routing/:agentName/providers/:provider` | Disconnect provider |
| POST | `/api/v1/routing/:agentName/providers/deactivate-all` | Deactivate all providers |
| GET | `/api/v1/routing/:agentName/tiers` | Get all tier assignments |
| PUT | `/api/v1/routing/:agentName/tiers/:tier` | Override tier model |
| DELETE | `/api/v1/routing/:agentName/tiers/:tier` | Clear tier override |
| POST | `/api/v1/routing/:agentName/tiers/reset-all` | Reset all overrides to auto-assigned |
| GET/PUT/DELETE | `/api/v1/routing/:agentName/tiers/:tier/fallbacks` | Tier fallback chain |
| GET | `/api/v1/routing/:agentName/complexity/status` | Complexity routing enabled? |
| POST | `/api/v1/routing/:agentName/complexity/toggle` | Toggle complexity routing |
| GET | `/api/v1/routing/:agentName/specificity` | All specificity assignments |
| PUT | `/api/v1/routing/:agentName/specificity/:category` | Set category model |
| POST | `/api/v1/routing/:agentName/specificity/:category/toggle` | Enable/disable category |
| DELETE | `/api/v1/routing/:agentName/specificity/:category` | Remove category config |
| PUT/DELETE | `/api/v1/routing/:agentName/specificity/:category/fallbacks` | Category fallbacks |
| POST | `/api/v1/routing/:agentName/specificity/reset-all` | Reset all specificity config |
| GET/POST/PUT/DELETE | `/api/v1/routing/:agentName/custom-providers` | Custom provider CRUD |
| POST | `/api/v1/routing/:agentName/custom-providers/probe` | Test custom provider connectivity |
| GET/POST/PUT/PATCH/DELETE | `/api/v1/routing/:agentName/header-tiers` | Header-based tier rules |
| GET | `/api/v1/routing/:agentName/seen-headers` | Headers seen in recent requests |
| GET | `/api/v1/routing/:agentName/available-models` | Models available from connected providers |
| POST | `/api/v1/routing/:agentName/refresh-models` | Re-fetch models from provider APIs |
| POST | `/api/v1/routing/ollama/sync` | Sync Ollama local model list |
| GET | `/api/v1/routing/pricing-health` | OpenRouter pricing cache health |
| POST | `/api/v1/routing/pricing/refresh` | Manually refresh pricing cache |
| POST | `/api/v1/routing/:agentName/copilot/device-code` | Start Copilot device-code flow |
| POST | `/api/v1/routing/:agentName/copilot/poll-token` | Poll for Copilot OAuth token |

## Notifications

| Method | Route | Description |
|--------|-------|-------------|
| GET/POST | `/api/v1/notifications` | List / create notification rules |
| PATCH/DELETE | `/api/v1/notifications/:id` | Update / delete rule |
| GET/POST/DELETE | `/api/v1/notifications/email-provider` | Email provider config |
| GET/POST | `/api/v1/notifications/notification-email` | View / send test email |
| POST | `/api/v1/notifications/trigger-check` | Manually trigger threshold check |
| GET | `/api/v1/notifications/logs` | Notification delivery history |

## Proxy (Agent Auth)

| Method | Route | Auth | Description |
|--------|-------|------|-------------|
| POST | `/v1/chat/completions` | Bearer mnfst_* | OpenAI chat completions proxy |
| POST | `/v1/responses` | Bearer mnfst_* | OpenAI Responses API proxy |
| POST | `/api/v1/routing/resolve` | Bearer mnfst_* | Model resolution (without proxying) |

## Real-Time

| Method | Route | Description |
|--------|-------|-------------|
| GET | `/api/v1/events` | SSE stream for real-time dashboard updates |

## See Also

- [authentication.md](authentication.md) — how auth works for each route type
- [routing.md](routing.md) — routing config semantics
- [data-flow.md](data-flow.md) — proxy endpoint lifecycle
