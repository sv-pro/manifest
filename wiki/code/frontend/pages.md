# frontend/pages/

Route-level page components. Each page corresponds to a top-level dashboard section.

## Page Inventory

| Page | Route | Description |
|------|-------|-------------|
| `Login.tsx` | `/login` | Email/password login form + social OAuth buttons |
| `Register.tsx` | `/register` | New account registration |
| `ResetPassword.tsx` | `/reset-password` | Password reset flow (request + confirm token) |
| `Workspace.tsx` | `/` | Agent grid — list all agents, create new agent, agent type selection |
| `Overview.tsx` | `/:agent/overview` | Agent dashboard: message/token/cost cards, tier sparklines, recent messages |
| `MessageLog.tsx` | `/:agent/messages` | Paginated message log with routing badges, tier colors, specificity labels, feedback |
| `Settings.tsx` | `/:agent/settings` | Agent name, platform, API key management |
| `Routing.tsx` | `/:agent/routing` | Tier assignments, provider connections, specificity config, header-tier rules |
| `RoutingTierCard.tsx` | (sub-component) | Tier slot editor with model picker, fallback chain, complexity toggle |
| `Limits.tsx` | `/:agent/limits` | Notification rules CRUD, email provider config |
| `Account.tsx` | `/account` | User profile, session info, password change |
| `ModelPrices.tsx` | `/model-prices` | Full model pricing table sourced from OpenRouter cache |
| `FreeModels.tsx` | `/free-models` | Zero-cost model list per provider |
| `Help.tsx` | `/help` | Getting started docs, agent setup guide |
| `Setup.tsx` | `/setup` | First-run admin account wizard (shown before login) |
| `NotFound.tsx` | `*` | 404 page |

## Shared Patterns

- Pages fetch data via service functions from `services/api/`
- Loading states use SolidJS `createResource` or `createSignal`
- Toast notifications via `toast-store.ts`
- All authenticated pages are wrapped in `AuthGuard` → `AgentGuard`

## See Also

- [components.md](components.md) — reusable components used by pages
- [services.md](services.md) — API functions called by pages
