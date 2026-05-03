# shared/specificity.ts

Defines the canonical set of specificity category IDs used for task-type routing.

## Exports

```typescript
export const SPECIFICITY_CATEGORIES = [
  'coding', 'web_browsing', 'data_analysis', 'image_generation',
  'video_generation', 'social_media', 'email_management',
  'calendar_management', 'trading',
] as const;

export type SpecificityCategory = (typeof SPECIFICITY_CATEGORIES)[number];
```

## Consumers

- **Backend**: `SpecificityAssignment` entity uses `SpecificityCategory` as a column type; `SpecificityDetector` iterates over categories; `DIMENSION_MAP` keys are `SpecificityCategory`
- **Frontend**: category labels rendered in Routing config UI; `SpecificityAssignment` API types

## See Also

- [../../specificity.md](../../specificity.md) — detection algorithm
- [../backend/scoring.md](../backend/scoring.md) — specificity-detector.ts
