# shared/tiers.ts

Defines the four routing complexity tiers plus the `default` slot.

## Exports

```typescript
export const TIERS = ['simple', 'standard', 'complex', 'reasoning'] as const;
export type Tier = (typeof TIERS)[number];

export const TIER_DESCRIPTIONS: Record<Tier, string> = {
  simple:    'Factual lookups, short transforms, trivial completions',
  standard:  'Typical multi-step tasks, moderate reasoning',
  complex:   'Multi-document synthesis, hard code generation, long outputs',
  reasoning: 'Chain-of-thought problems, formal logic, deep analysis',
};
```

## Usage

- **Backend**: `TierAssignment` entity has a slot per tier; `scoreRequest()` returns a `Tier`; `TierAutoAssignService` fills slots
- **Frontend**: tier labels in the Routing config UI; tier badges in the Messages log

## See Also

- [tier-colors.md](tier-colors.md) — UI colors
- [../../routing.md](../../routing.md) — tier assignment semantics
- [../../scoring.md](../../scoring.md) — how tiers are determined
