# manifest-frontend

SolidJS SPA dashboard. Compiled by Vite; served by NestJS in production via `@nestjs/serve-static`.

## Modules

| Module | Page | Description |
|--------|------|-------------|
| `pages/` | [pages.md](pages.md) | Route-level page components (23 pages) |
| `components/` | [components.md](components.md) | Reusable UI components (60+ files) |
| `services/` | [services.md](services.md) | API wrappers, formatters, routing helpers, theme |
| `layouts/` | — | `AuthLayout` (guards + sidebar) and `GuestLayout` (centered card) |
| `styles/` | — | Global CSS variables, theme tokens, component styles |

## Entry Point: index.tsx

Sets up the SolidJS router with two layout trees:

```
GuestLayout
  ├── /login → Login
  ├── /register → Register
  └── /reset-password → ResetPassword

AuthLayout (guarded by AuthGuard)
  ├── / → Workspace (agent grid)
  └── /:agentName (guarded by AgentGuard)
        ├── /overview → Overview
        ├── /messages → MessageLog
        ├── /settings → Settings
        ├── /routing → Routing
        ├── /limits → Limits
        ├── /account → Account
        ├── /model-prices → ModelPrices
        ├── /free-models → FreeModels
        └── /help → Help
```

All page components are **lazy-loaded** (`lazy(() => import(…))`).

## Key Conventions

- **No external CDN resources** — all fonts/icons are self-hosted in `public/`
- **credentials: include** on all fetch calls (Better Auth cookie forwarding)
- **SolidJS reactivity** — use signals and stores, not React hooks
- Vite dev server on `:3000` proxies `/api` and `/otlp` to backend `:3001`

## See Also

- [../../architecture.md](../../architecture.md) — dev mode proxy setup
- [../../authentication.md](../../authentication.md) — AuthGuard, GuestGuard behavior
