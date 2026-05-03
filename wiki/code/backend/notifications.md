# backend/notifications/

Alert threshold rules, email dispatch, and delivery logging.

## Files

| File | Description |
|------|-------------|
| `notifications.controller.ts` | Full CRUD for rules + email provider + test/trigger endpoints |
| `alert-scenarios.service.ts` | Evaluates whether a rule's threshold is breached, with debounce |
| `email-provider-config.service.ts` | Read/write encrypted email provider credentials |
| `email-provider-validation.service.ts` | Validates credentials before saving (test send) |
| `limit-check.service.ts` | Queries `agent_messages` for current metric value in window |
| `notification-cron.service.ts` | `@Cron(EVERY_HOUR)` — runs threshold checks for all enabled rules |
| `notification-email.service.ts` | Dispatches email via Mailgun / Resend / SMTP |
| `notification-log.service.ts` | Appends `NotificationLog` rows on send/fail |
| `emails/` | React Email templates rendered server-side |
| `dto/` | Input validation DTOs |

## Cron Cycle

```
EVERY_HOUR:
  For each enabled NotificationRule:
    ├── LimitCheckService.check(rule)    → current value
    ├── AlertScenariosService.evaluate() → should fire?
    └── YES:
          ├── NotificationEmailService.send()
          └── NotificationLogService.record()
```

POST `/api/v1/notifications/trigger-check` runs the cycle immediately (for testing).

## Email Provider Priority

1. Tenant's own `EmailProviderConfig` (Mailgun / Resend / SMTP)
2. Global env vars: `MAILGUN_API_KEY` + `MAILGUN_DOMAIN`
3. Fail with a clear error if neither is configured

## See Also

- [../../notifications.md](../../notifications.md) — concept page
- [../entities.md](entities.md) — NotificationRule, NotificationLog, EmailProviderConfig entities
