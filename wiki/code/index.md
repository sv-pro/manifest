# Code Wiki

Hierarchical summaries of the Manifest monorepo. Each package has an `index.md`; each major module has its own page.

## Packages

| Package | npm name | Description |
|---------|----------|-------------|
| [shared/](shared/index.md) | `manifest-shared` | Canonical types and constants shared between backend and frontend |
| [backend/](backend/index.md) | `manifest-backend` | NestJS API server — routing engine, proxy, analytics, auth |
| [frontend/](frontend/index.md) | `manifest-frontend` | SolidJS SPA dashboard |
| [manifest/](manifest/index.md) | `manifest` | Code-free version shell; holds canonical Docker image version |

## Cross-Package Conventions

- **Shared types first**: if a type or constant is used by both backend and frontend, it lives in `manifest-shared`. Never duplicate.
- **Import path**: backend uses `import { … } from 'manifest-shared'`; frontend uses a direct relative import from `packages/shared/src/`.
- **Strict TypeScript**: all packages use `strict: true`. No `any` without a comment.
- **No synchronize**: TypeORM `synchronize` is permanently `false` in backend. All schema changes go through versioned migrations.
- **100% coverage**: every new source line must be covered by tests. See [development.md](../development.md).

## Key Cross-Package Flows

```
shared/providers.ts
  ├── backend: PROVIDER_REGISTRY (re-export) → proxy adapters, model discovery
  └── frontend: PROVIDERS → routing UI, provider tiles

shared/specificity.ts
  ├── backend: SPECIFICITY_CATEGORIES → detector, DB enum
  └── frontend: category labels, specificity config UI

shared/tiers.ts
  ├── backend: tier slot names → TierAssignment entity
  └── frontend: tier labels, tier color mapping
```
