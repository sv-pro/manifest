# backend/model-prices/

Exposes the OpenRouter pricing cache as an authenticated endpoint.

## Files

- `model-prices.controller.ts` — `GET /api/v1/model-prices`
- `model-prices.module.ts`
- `model-prices.service.ts` — delegates to `ModelPricingCacheService` from `database/`

## What It Returns

All models known to the OpenRouter cache, attributed to their real provider using `OPENROUTER_PREFIX_TO_PROVIDER`. Community/unsupported vendors are grouped under "OpenRouter".

Displayed in the **Model Prices** page of the dashboard.

## See Also

- [database.md](database.md) — PricingSyncService (the actual cache)
- [../../models.md](../../models.md) — pricing source and priority
