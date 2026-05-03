# backend/github/

Proxies the GitHub star count for the Manifest repository.

## Files

- `github.controller.ts` — `GET /api/v1/github/stars` (`@Public()`)
- `github.module.ts`

Returns `{ stars: number }`. The value is cached to avoid hitting GitHub's unauthenticated rate limit on every dashboard load. Displayed in the Manifest marketing header.

## See Also

- [../../api.md](../../api.md) — full API reference
