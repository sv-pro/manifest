# backend/scoring/

Complexity scoring engine and specificity detector. Determines which routing tier each request gets.

## Files

| File | Description |
|------|-------------|
| `index.ts` | `scoreRequest(messages, config)` — main entry point; returns `{ tier, score, signals }` |
| `config.ts` | `DEFAULT_CONFIG` — 30 dimension definitions, tier boundaries, confidence thresholds |
| `types.ts` | `ScoringConfig`, `DimensionResult`, `ScoringResult` types |
| `specificity-detector.ts` | `SpecificityDetector.detect(messages, tools)` — returns winning category or null |
| `specificity-signals.ts` | `SpecificitySignal[]` per category — structured signal definitions |
| `specificity-weights.ts` | Per-category weight tuning constants |
| `envelope-peeler.ts` | Extract user-facing messages from an OpenAI-format payload |
| `text-extractor.ts` | Strip markdown, code fences, tool results from message content |
| `scan-messages.ts` | Runs keyword scan across all messages; returns match counts |
| `keyword-trie.ts` | Aho-Corasick trie for O(n) keyword matching |
| `keywords.ts` | All keyword lists for complexity dimensions |
| `momentum.ts` | Session rolling score; `applyMomentum(score, history)` |
| `overrides.ts` | Hard-coded score overrides (e.g. `X-Manifest-Tier` header) |
| `sigmoid.ts` | `sigmoid(x, k)` — maps unbounded raw score to `[0, 1]` |

## keywords/ subdirectory

Per-specificity-category keyword lists:

```
complexity.ts          — formalLogic, codeGeneration, mathematicalReasoning, …
coding.ts              — (in keywords/ or keywords.ts)
web-browsing.ts
data-analysis.ts
image-generation.ts
video-generation.ts
social-media.ts
email-management.ts
calendar-management.ts
trading.ts
```

## dimensions/ subdirectory

Three dimension scorer files:

| File | Description |
|------|-------------|
| `keyword-dimensions.ts` | Scores each keyword dimension from trie match counts |
| `structural-dimensions.ts` | Scores code block count, nesting depth, constraint count |
| `contextual-dimensions.ts` | Scores turn count, tool presence, system prompt length |

## Scoring Pipeline

```typescript
const result = scoreRequest(messages, DEFAULT_CONFIG);
// result.tier: 'simple' | 'standard' | 'complex' | 'reasoning'
// result.score: 0–1 normalized
// result.signals: Record<dimension, rawScore>
```

## See Also

- [../../scoring.md](../../scoring.md) — concept page with full algorithm description
- [../../specificity.md](../../specificity.md) — specificity detection concept
- [routing.md](routing.md) — how the score feeds into model selection
