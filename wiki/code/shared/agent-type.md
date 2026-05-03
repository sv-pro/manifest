# shared/agent-type.ts

Agent classification constants: categories, platforms, labels, icons.

## Exports

```typescript
// Two agent categories
AGENT_CATEGORIES = ['personal', 'app']

// Seven platforms
AGENT_PLATFORMS = [
  'openclaw', 'hermes', 'openai-sdk',
  'vercel-ai-sdk', 'langchain', 'curl', 'other'
]

// Human-readable labels
CATEGORY_LABELS: { personal: 'Personal AI Agent', app: 'App AI SDK' }
PLATFORM_LABELS: { openclaw: 'OpenClaw', 'openai-sdk': 'OpenAI SDK', … }

// Icon paths (relative to frontend public/)
PLATFORM_ICONS: { openclaw: '/icons/openclaw.png', … }

// Which platforms belong to which category
PLATFORMS_BY_CATEGORY: {
  personal: ['openclaw', 'hermes', 'other'],
  app: ['openai-sdk', 'vercel-ai-sdk', 'langchain', 'other'],
}
```

### `platformIcon(platform, category): string | undefined`

Helper that returns the icon path for a platform. Handles the `other` platform's dual icon (agent vs. app variant).

## See Also

- [../backend/entities.md](../backend/entities.md) — Agent entity `platform` column
- [../frontend/components.md](../frontend/components.md) — AgentTypeGrid
