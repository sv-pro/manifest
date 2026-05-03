# Notifications

Alert system that monitors agent metrics and fires email notifications when thresholds are breached.

## Concepts

- **NotificationRule**: a threshold rule owned by a user (tenant-scoped). Defines the metric, comparator, threshold, time window, and enabled state.
- **NotificationLog**: append-only delivery record for each fired rule. Tracks status (sent/failed), timestamps, error message.
- **EmailProviderConfig**: stored email provider credentials for the tenant. Supports Mailgun, Resend, or SMTP.

## Supported Metrics

- `cost_usd` — total spend in window
- `tokens_input` / `tokens_output` — token counts
- `message_count` — number of requests
- `error_rate` — percentage of failed requests

## Threshold Check Cycle

`NotificationCronService` runs `@Cron(CronExpression.EVERY_HOUR)`:

```
For each enabled NotificationRule:
  ├── Query agent_messages for the window (last N hours/days)
  ├── Compute current metric value
  ├── AlertScenariosService.evaluate(rule, value)
  │     ├── > threshold? → should_fire
  │     └── Recently fired? → debounce (skip)
  └── If should_fire:
        ├── NotificationEmailService.send(rule, context)
        └── NotificationLogService.record(rule, result)
```

`POST /api/v1/notifications/trigger-check` runs this cycle on-demand (useful for testing rules).

## Email Providers

Three backends, configured per-tenant via `EmailProviderConfig`:

| Provider | Required fields |
|----------|----------------|
| Mailgun | `apiKey`, `domain` |
| Resend | `apiKey` |
| SMTP | `host`, `port`, `username`, `password` |

Credentials are encrypted at rest (AES-256-GCM, same as provider API keys). `EmailProviderValidationService` validates credentials before saving.

Global fallback: if no per-tenant config, uses `MAILGUN_API_KEY` + `MAILGUN_DOMAIN` env vars.

## Email Templates

`notifications/emails/` contains React Email templates rendered server-side before sending. Templates receive structured context (rule details, current value, agent name).

## Alert Scenarios

`AlertScenariosService` handles:
- **Absolute threshold** — value > N
- **Rate of change** — value increased by N% vs. previous window
- **Debounce** — don't re-fire within `cool_down_minutes` of last notification

## API

| Route | Description |
|-------|-------------|
| `GET /api/v1/notifications` | List all rules |
| `POST /api/v1/notifications` | Create rule |
| `PATCH /api/v1/notifications/:id` | Update rule |
| `DELETE /api/v1/notifications/:id` | Delete rule |
| `GET/POST/DELETE /api/v1/notifications/email-provider` | Email provider config |
| `POST /api/v1/notifications/trigger-check` | Manual trigger |
| `GET /api/v1/notifications/logs` | Delivery history |

## See Also

- [entities.md](entities.md) — NotificationRule, NotificationLog, EmailProviderConfig entities
- [development.md](development.md) — environment variables for email
- [code/backend/notifications.md](code/backend/notifications.md) — module internals
