# backend/analytics/

Dashboard analytics: aggregation queries, timeseries, message log, savings, and per-agent stats.

## Controllers

| Controller | Prefix | Endpoints |
|-----------|--------|-----------|
| `overview.controller.ts` | `/api/v1/overview` | Dashboard summary (messages, costs, tokens, recent activity) |
| `tokens.controller.ts` | `/api/v1/tokens` | Token usage timeseries |
| `costs.controller.ts` | `/api/v1/costs` | Cost timeseries |
| `messages.controller.ts` | `/api/v1/messages` | Paginated message log + detail + feedback + miscategorized |
| `agents.controller.ts` | `/api/v1/agents` | Agent CRUD + sparklines + duplication |
| `savings.controller.ts` | `/api/v1/savings` | Cost savings vs. baseline, timeseries, baseline candidates |
| `agent-analytics.controller.ts` | `/api/v1/agent/:agentName` | Per-agent usage and costs |

## Services

| Service | Description |
|---------|-------------|
| `aggregation.service.ts` | Token/cost/message aggregate counts for the overview card |
| `timeseries-queries.service.ts` | Time-bucketed timeseries for tokens, costs, and recent activity |
| `messages-query.service.ts` | Paginated message log with filtering, sorting, and shared projection |
| `message-details.service.ts` | Full message detail: tokens breakdown, headers, tool executions, latency |
| `message-feedback.service.ts` | Read/write routing feedback on individual messages |
| `agent-analytics.service.ts` | Per-agent token and cost aggregates |
| `agent-duplication.service.ts` | Clone agent routing config to a new agent |
| `agent-lifecycle.service.ts` | Agent CRUD, API key rotation, soft delete |
| `savings-query.service.ts` | Cost savings calculation vs. a configurable baseline model |
| `specificity-feedback.service.ts` | Read/write specificity miscategorization flags |
| `query-helpers.ts` | **Critical**: `selectMessageRowColumns()`, `addTenantFilter()`, `downsample()`, `computeTrend()` |

## Shared Projection Contract

`selectMessageRowColumns(qb)` in `query-helpers.ts` is the **single source of truth** for which columns `MessageTable` / `ModelCell` can read. All message-list endpoints **must** call this helper. A spec test (`query-helpers.spec.ts`) pins the required alias set and fails loudly if a column is dropped.

```typescript
// Pattern:
const qb = this.agentMessageRepo.createQueryBuilder('m');
selectMessageRowColumns(qb);  // always first
addTenantFilter(qb, userId);  // always second
// then: endpoint-specific addSelect, where, orderBy, …
```

## Query Patterns

All analytics queries use TypeORM `createQueryBuilder()`. Key helpers from `postgres-sql.ts`:
- `bucketColumn(interval)` — `date_trunc` expression for time bucketing
- `castInterval(param)` — `CAST(:param AS interval)` for window filters

Raw `DataSource.query()` is used only in the notification cron and database seeder.

## See Also

- [../../entities.md](../../entities.md) — agent_messages key columns
- [../../data-flow.md](../../data-flow.md) — how messages are written
- [../../multi-tenancy.md](../../multi-tenancy.md) — tenant filtering
