# frontend/components/

Reusable UI components. 60+ files covering guards, charts, modals, routing UI, and shared widgets.

## Guards & Layout

| Component | Description |
|-----------|-------------|
| `AuthGuard.tsx` | Checks session; redirects to `/login` if not authenticated |
| `GuestGuard.tsx` | Redirects authenticated users away from auth pages |
| `AgentGuard.tsx` | Checks that the `:agentName` param resolves to a valid agent for this user |

## Agent & Workspace

| Component | Description |
|-----------|-------------|
| `AgentTypeGrid.tsx` | Displays agent category + platform selection grid |
| `AgentTypeSelect.tsx` | Platform picker dropdown |
| `SetupModal.tsx` | Multi-step agent setup wizard (first-time flow) |
| `FrameworkSnippets.tsx` | Code snippets for configuring each SDK to use Manifest |

## Routing UI

| Component | Description |
|-----------|-------------|
| `CustomProviderForm.tsx` | Add/edit custom OpenAI-compatible provider |
| `FallbackList.tsx` | Drag-ordered fallback model chain editor |
| `HeaderTierRuleEditor.tsx` | Create/edit header-based tier routing rules |
| `ProviderTile.tsx` | Provider connection card with connect/disconnect action |

## Analytics & Charts

| Component | Description |
|-----------|-------------|
| `CostByModelTable.tsx` | Cost breakdown table grouped by model |
| `SparklineChart.tsx` | Small uPlot inline chart for tier sparklines |
| `TimeseriesChart.tsx` | Full-size uPlot chart for tokens/costs over time |
| `TierBadge.tsx` | Colored pill showing routing tier (simple/standard/complex/reasoning) |
| `ModelCell.tsx` | Renders model name + provider icon + tier badge in the message table |
| `AuthBadge.tsx` | Shows auth type (API key, subscription, local) |

## Notifications

| Component | Description |
|-----------|-------------|
| `EmailProviderModal.tsx` | Email provider config modal (Mailgun/Resend/SMTP) |
| `NotificationRuleForm.tsx` | Create/edit alert threshold rule |

## Shared

| Component | Description |
|-----------|-------------|
| `Header.tsx` | Top nav bar: user avatar, logout, theme toggle |
| `Sidebar.tsx` | Left nav sidebar with agent/page links |
| `Toast.tsx` | Toast notification rendered from `toast-store.ts` |
| `Pagination.tsx` | Page selector for message log |
| `Modal.tsx` | Generic modal wrapper |
| `CopyButton.tsx` | One-click clipboard copy with confirmation animation |

## See Also

- [pages.md](pages.md) — pages that compose these components
- [services.md](services.md) — API calls made from within components
