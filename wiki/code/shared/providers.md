# shared/providers.ts

**Single source of truth for all LLM provider definitions.**

## Exports

### `SHARED_PROVIDERS: readonly SharedProviderEntry[]`

Array of 17 provider entries. Each entry has:

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Canonical ID (e.g. `anthropic`, `gemini`) |
| `displayName` | string | Human-readable name (e.g. `Anthropic`, `Google`) |
| `aliases` | string[] | Alternative IDs that map to this entry |
| `openRouterPrefixes` | string[] | OpenRouter vendor prefixes for model attribution |
| `requiresApiKey` | boolean | Whether an API key must be provided |
| `localOnly` | boolean | Whether the provider runs on the user's machine |
| `color` | string | Brand hex color for UI |
| `keyPrefix` | string | Expected key prefix (e.g. `sk-ant-`) |
| `minKeyLength` | number | Minimum plausible key length |
| `keyPlaceholder` | string | Placeholder for the UI input |
| `tileOnly?` | boolean | Tile-only (no fixed endpoint; uses custom URL) |

### Derived Maps

```typescript
SHARED_PROVIDER_BY_ID          // Map<id, entry>
SHARED_PROVIDER_BY_ID_OR_ALIAS // Map<id|alias, entry>
```

### `LOCAL_SERVER_HINTS`

Setup hints for `ollama`, `lmstudio`, `llamacpp`: default port, CLI command, install URL, Docker bind notes.

### `normalizeProviderName(s: string): string`

Collapses whitespace/dots/underscores/hyphens for fuzzy name matching.

## Backend Re-Export

`packages/backend/src/common/constants/providers.ts` re-exports under historical backend-facing names:
- `PROVIDER_REGISTRY` = `SHARED_PROVIDERS`
- `PROVIDER_BY_ID`, `PROVIDER_BY_ID_OR_ALIAS`, `OPENROUTER_PREFIX_TO_PROVIDER`

## See Also

- [../../providers.md](../../providers.md) — concept page
- [../backend/routing.md](../backend/routing.md) — proxy adapters
- [../backend/model-discovery.md](../backend/model-discovery.md) — fetcher configs
