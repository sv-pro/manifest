# Providers

Manifest supports 17 LLM providers through a canonical shared registry.

## Canonical Registry

**Single source of truth**: `packages/shared/src/providers.ts` — `SHARED_PROVIDERS`.

The backend re-exports from `common/constants/providers.ts` under historical names:
- `PROVIDER_REGISTRY` — full list
- `PROVIDER_BY_ID` — map by canonical ID
- `PROVIDER_BY_ID_OR_ALIAS` — map by ID or alias
- `OPENROUTER_PREFIX_TO_PROVIDER` — OpenRouter prefix → display name

**Never hardcode provider names/IDs outside the shared package.**

## Provider List

| ID | Display Name | Local Only | Tile Only | Key Prefix |
|----|-------------|-----------|----------|-----------|
| `qwen` | Alibaba | — | — | `sk-` |
| `anthropic` | Anthropic | — | — | `sk-ant-` |
| `deepseek` | DeepSeek | — | — | `sk-` |
| `copilot` | GitHub Copilot | — | — | (OAuth) |
| `gemini` | Google | — | — | (no prefix) |
| `minimax` | MiniMax | — | — | `sk-` |
| `mistral` | Mistral | — | — | (no prefix) |
| `moonshot` | Moonshot (Kimi) | — | — | `sk-` |
| `llamacpp` | llama.cpp | ✓ | ✓ | — |
| `lmstudio` | LM Studio | ✓ | ✓ | — |
| `ollama` | Ollama | ✓ | — | — |
| `ollama-cloud` | Ollama Cloud | — | — | — |
| `openai` | OpenAI | — | — | `sk-` |
| `opencode-go` | OpenCode Go | — | — | (no prefix) |
| `openrouter` | OpenRouter | — | — | `sk-or-` |
| `xai` | xAI | — | — | `xai-` |
| `zai` | Z.ai | — | — | (no prefix) |

**Aliases** (also recognized): `alibaba` → `qwen`, `google` → `gemini`, `kimi` → `moonshot`, `llama.cpp` → `llamacpp`, `lm-studio` → `lmstudio`

## Local-Only Providers

`ollama`, `lmstudio`, `llamacpp` have `localOnly: true`. The proxy tags their messages `auth_type: 'local'` and they appear under the Local tab in the UI.

`MANIFEST_MODE=selfhosted` (or Docker) is required to connect providers via `http://` or private IP ranges. Cloud mode enforces HTTPS public endpoints.

## Tile-Only Providers

`lmstudio` and `llamacpp` have `tileOnly: true` — they appear as tiles in the UI but have no fixed proxy endpoint. Once connected, the proxy routes through a `custom:<uuid>` path using the user-entered base URL. Provider endpoint sanity tests skip these.

## API Key Encryption

Provider API keys are encrypted at rest using **AES-256-GCM**. `ProviderKeyService` derives the encryption key from `BETTER_AUTH_SECRET`. Keys are decrypted in-memory only when needed for a proxy request and are never returned by the API after initial connection.

## GitHub Copilot (OAuth)

Copilot uses device-code OAuth (not an API key):
1. `POST /api/v1/routing/:agentName/copilot/device-code` — initiate device flow
2. `POST /api/v1/routing/:agentName/copilot/poll-token` — poll for token

## MiniMax / OpenAI OAuth

Both providers support OAuth flows via `routing/oauth/`. MiniMax uses a device-code pattern; OpenAI OAuth follows a standard redirect flow.

## Local Server Hints

`packages/shared/src/providers.ts` also exports `LOCAL_SERVER_HINTS` — setup commands, install URLs, Docker bind notes, and CLI flags for Ollama, LM Studio, and llama.cpp. Consumed by the frontend provider detail view.

## Adding a New Provider

See [development.md](development.md#adding-a-new-provider).

## See Also

- [models.md](models.md) — model discovery per provider
- [routing.md](routing.md) — how providers are selected
- [code/shared/providers.md](code/shared/providers.md) — shared registry file
- [code/backend/routing.md](code/backend/routing.md) — proxy adapters
