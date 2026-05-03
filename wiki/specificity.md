# Specificity Routing

Specificity routing detects the *task type* of a request (coding, trading, image generation, etc.) and routes it to a model specialized for that domain, overriding the generic complexity tier.

## The 9 Categories

Defined in `packages/shared/src/specificity.ts`:

| Category | Typical requests |
|----------|-----------------|
| `coding` | Write code, debug, refactor, explain code, generate tests |
| `web_browsing` | Fetch URL, search web, scrape page, navigate browser |
| `data_analysis` | Analyze CSV, plot chart, run SQL, statistical summary |
| `image_generation` | Generate image, create illustration, edit photo |
| `video_generation` | Create video, animate, generate clip |
| `social_media` | Write tweet/post, schedule content, manage engagement |
| `email_management` | Draft email, reply to thread, categorize inbox |
| `calendar_management` | Schedule meeting, create event, find free slot |
| `trading` | Buy/sell stock, execute trade, fetch market data, backtest |

## Detection Algorithm

`SpecificityDetector` in `scoring/specificity-detector.ts` runs on the **last user message** (not the full history), because specificity is about the current intent.

```
Last user message + tool names in payload
  ▼
1. Keyword scan per category (via KeywordTrie)
     Each category has a list of keywords in scoring/keywords/{category}.ts
  ▼
2. Tool name heuristics (TOOL_NAME_PATTERNS)
     Tool names like "browser_*", "execute_python", "place_order" are strong signals
  ▼
3. Weighted sum per category → raw specificity scores
  ▼
4. Session bias — if previous turns in this session had a strong category, apply momentum
  ▼
5. Specificity penalty — penalize categories that are weak relative to the winner
  ▼
6. Threshold check — is the winning category score above MIN_CONFIDENCE?
     YES → return category
     NO  → return null (fall through to complexity scoring)
```

## Signals per Category

`specificity-signals.ts` defines `SpecificitySignal[]` for each category — structured signal definitions with:
- `type`: `keyword` | `tool_name` | `structural`
- `weight`: contribution to category score
- `pattern`: keyword or regex

`specificity-weights.ts` holds per-category weight tuning, allowing some categories to be more conservative (high threshold) or aggressive (low threshold) in detection.

## Enabling Specificity for an Agent

Specificity routing is **opt-in per category per agent**:

```
PUT /api/v1/routing/:agentName/specificity/:category
{
  "enabled": true,
  "modelId": "claude-opus-4-7",
  "providerId": "anthropic"
}
```

Only categories explicitly enabled are checked. An agent can enable all 9 or just one.

## Miscategorization Feedback

Users can flag a message as miscategorized via `PATCH /api/v1/messages/:id/miscategorized`. This data is tracked in `specificity_feedback` and visible in the Messages log, but does not currently auto-retrain the detector.

## Adding a New Category

1. Add ID to `SPECIFICITY_CATEGORIES` in `packages/shared/src/specificity.ts`
2. Add keyword file at `scoring/keywords/{category}.ts`
3. Add dimension to `DEFAULT_CONFIG.dimensions` in `scoring/config.ts`
4. Add to `DIMENSION_MAP` in `scoring/specificity-detector.ts`
5. Optionally add to `TOOL_NAME_PATTERNS`
6. Add `StageDef` to frontend `packages/frontend/src/services/providers.ts`
7. Add coverage test prompts to `scoring/__tests__/specificity-coverage.spec.ts`

The `specificity_assignments` table and UI handle new categories automatically.

## See Also

- [routing.md](routing.md) — how specificity fits into routing resolution
- [scoring.md](scoring.md) — complexity scoring (the fallback path)
- [code/backend/scoring.md](code/backend/scoring.md) — scorer module
- [code/shared/specificity.md](code/shared/specificity.md) — category type definitions
