# frontend/services/

API wrappers, utility services, and state helpers.

## API Layer (services/api/)

`api.ts` re-exports all sub-modules:

| Module | Description |
|--------|-------------|
| `api/core.ts` | Base HTTP client wrapper (`apiFetch`, handles credentials, error parsing) |
| `api/agents.ts` | Agent CRUD, key rotation, duplication |
| `api/messages.ts` | Message log, detail, feedback, miscategorized |
| `api/analytics.ts` | Overview, tokens, costs, savings |
| `api/routing.ts` | Providers, tiers, fallbacks, complexity toggle, available models |
| `api/specificity.ts` | Specificity assignments, toggle, reset |
| `api/notifications.ts` | Notification rules + email provider |
| `api/oauth.ts` | OAuth popup helpers (OpenAI, MiniMax, Copilot flows) |
| `api/free-models.ts` | Free model list |

## Provider & Routing Services

| File | Description |
|------|-------------|
| `providers.ts` | `PROVIDERS` array (from shared) + `StageDef` for each provider's UI stage |
| `routing-params.ts` | Reactive state helpers for routing config URL params |
| `routing-utils.ts` | Pure helpers for tier/specificity data transformation |
| `framework-snippets.ts` | Code snippet templates for each SDK platform |
| `oauth-popup.ts` | Opens OAuth window, polls for completion, returns token |

## Utilities

| File | Description |
|------|-------------|
| `auth-client.ts` | `createAuthClient()` from `better-auth/solid`. Used for session checks, logout. |
| `formatters.ts` | `formatCost(usd)`, `formatTokens(n)`, `formatDate(ts)` |
| `chart-utils.ts` | uPlot series configuration, axis formatters, downsample helpers |
| `theme.ts` | Read/write dark/light theme preference (`localStorage` + CSS class) |
| `toast-store.ts` | Global toast notification signal: `showToast(message, type)` |
| `provider-utils.ts` | Provider icon lookup, color lookup from shared registry |

## Patterns

All `apiFetch` calls include `credentials: 'include'` for Better Auth cookie forwarding. Errors are thrown as `ApiError` objects with `status` and `message` fields, caught by page components.

## See Also

- [pages.md](pages.md) — where service functions are called
- [../../api.md](../../api.md) — backend endpoint reference
