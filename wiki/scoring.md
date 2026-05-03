# Complexity Scoring

The scoring engine analyzes each request and assigns a complexity tier (`simple` / `standard` / `complex` / `reasoning`). It lives in `packages/backend/src/scoring/`.

## How It Works

```
Request messages
  ▼
EnvelopePeeler — extract user-facing messages from the payload
  ▼
TextExtractor  — strip markdown, code fences, tool results
  ▼
30 dimensions evaluated in parallel:
  ├── Keyword dimensions    — trie-based keyword matching
  ├── Structural dimensions — code blocks, nesting depth, constraint count
  └── Contextual dimensions — message count, role distribution, tool presence
  ▼
Each dimension → raw score → sigmoid normalization → weighted sum
  ▼
Momentum adjustment (session stickiness)
  ▼
Tier boundaries → final tier
```

## Dimensions (30 total)

Defined in `scoring/config.ts` as `DEFAULT_CONFIG.dimensions`. Key groups:

**Keyword-based** (detected via `KeywordTrie` for O(n) scanning):
- `formalLogic` — deductive reasoning, proofs, logic puzzles
- `codeGeneration` — write/implement/refactor keywords
- `mathematicalReasoning` — calculus, proofs, statistical modeling
- `researchSynthesis` — compare, analyze, literature review
- `creativeWriting` — narrative, storytelling, world-building
- `adversarialPrompting` — jailbreak attempts → elevated tier
- … and ~15 more

**Structural** (computed from message structure):
- `tokenCount` — raw message length proxy
- `nestedListDepth` — bullet/indent depth signals structured output needed
- `codeBlockCount` — number of code fences
- `constraintCount` — "must", "should not", "exactly N" constraint phrases

**Contextual** (conversation-level signals):
- `turnCount` — long conversations bias toward complex
- `toolPresence` — tool_call / function_call in payload
- `systemPromptLength` — long system prompts = complex use case

## Keyword Trie

`KeywordTrie` (in `scoring/keyword-trie.ts`) builds an Aho-Corasick style trie over all keyword lists at startup. Scanning a message is O(message length) regardless of keyword count — critical for low-latency routing.

## Sigmoid Normalization

Raw dimension scores are unbounded. Each is passed through a sigmoid to produce a `[0, 1]` value before weighting:

```typescript
// sigmoid.ts
sigmoid(x, k = 1) = 1 / (1 + exp(-k * x))
```

The `k` parameter (steepness) is tunable per dimension in `config.ts`.

## Tier Boundaries

After computing the weighted sum `S ∈ [0, 1]`, the tier is determined by thresholds in `DEFAULT_CONFIG`:

```
S < threshold.simple   → simple
S < threshold.standard → standard
S < threshold.complex  → complex
S >= threshold.complex → reasoning
```

## Score Overrides

`scoring/overrides.ts` defines hard-coded override rules that bypass the soft scoring — e.g. any message containing an explicit `X-Manifest-Tier` header skips scoring entirely.

## Momentum

`scoring/momentum.ts` maintains a rolling session score. If recent turns scored high, the current turn's tier is nudged upward. This prevents the session from oscillating between tiers on alternating long/short messages.

## See Also

- [routing.md](routing.md) — how the tier feeds into model selection
- [specificity.md](specificity.md) — category detection (parallel to scoring)
- [code/backend/scoring.md](code/backend/scoring.md) — module internals
