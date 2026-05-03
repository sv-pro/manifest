# shared/subscription/

Configuration and helpers for subscription-based LLM providers (e.g. Claude.ai, GitHub Copilot).

## Files

| File | Description |
|------|-------------|
| `configs.ts` | Per-provider subscription configs: supported tiers, model caps, rate limit hints |
| `types.ts` | `SubscriptionCapabilities` and `SubscriptionProviderConfig` types |
| `helpers.ts` | Utility functions for subscription tier validation |
| `index.ts` | Re-exports all of the above |

## Concepts

A **subscription provider** authenticates via OAuth (device-code or redirect flow) rather than an API key. The subscription config tells the routing layer what capabilities are available at what plan level — relevant for tier auto-assignment and the Anthropic subscription probe.

## See Also

- [../../providers.md](../../providers.md) — GitHub Copilot and Anthropic subscription sections
- [../backend/model-discovery.md](../backend/model-discovery.md) — AnthropicSubscriptionProbeService
