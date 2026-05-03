# backend/sse/

Server-Sent Events stream for real-time dashboard updates.

## Files

- `sse.controller.ts` — `GET /api/v1/events` (requires session auth)
- `sse.module.ts`

## Behavior

When a new `AgentMessage` is persisted by `ProxyMessageRecorder`, the `IngestEventBus` (in `common/services/`) emits an in-process event. `SseController` fans that event out to all connected SSE clients for the current user's tenant.

The frontend uses this to refresh Overview sparklines and the Messages log without polling.

## Connection Lifecycle

1. Client opens `GET /api/v1/events` with `EventSource` or `fetch` with `Accept: text/event-stream`
2. Server holds the connection open and streams `data: {...}\n\n` on each new message
3. Client reconnects automatically (built into EventSource API)

## See Also

- [common.md](common.md) — IngestEventBus
- [routing.md](routing.md) — ProxyMessageRecorder that triggers events
