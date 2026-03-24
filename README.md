# foxyBackend

Breezit AI-powered wedding/event vendor platform backend.

## Stack

Node.js 20, Express, TypeScript, MongoDB/Mongoose, Vitest (auth), Jest (admin)

## Module Structure

| Module | Port | Package | Purpose |
|--------|------|---------|---------|
| `auth/` | 3000 | standalone | Vendor/customer APIs, AI pipeline, calendar, marketplace |
| `admin/` | 3001 | standalone | Admin dashboard APIs, user management, analytics |
| `common/` | — | `@foxywedding/common` | Shared errors, middleware, types (in auth/node_modules) |

## Key Integrations

| Service | Purpose | Key Files |
|---------|---------|-----------|
| **Nylas** | Email/calendar sync (Google, Outlook) | `auth/src/modules/common/gmail/gmailController.ts`, `auth/src/modules/vendor/calendar/` |
| **ORQ.ai** | AI LLM orchestration (multi-agent pipeline) | `auth/src/modules/ai/orq/orqController.ts` |
| **Retell** | AI voice calls | `admin/src/modules/retell/` |
| **Twilio** | SMS send/receive | `auth/src/modules/chat/phoneMessage/` |
| **Stripe** | Payments, subscriptions | `auth/src/modules/vendor/`, `admin/src/modules/payments/` |
| **Slack** | Error notifications (#ai-errors) | `auth/src/modules/common/error/slackNotificationService.ts` |
| **Cal.com** | Appointment scheduling | `auth/src/modules/ai/utils/extract-email-data.utils.ts` |
| **MongoDB** | Primary database | Mongoose models throughout |
| **AWS S3** | File storage | `auth/src/modules/common/` |

## Auth Module Source Layout

```
auth/src/
├── app.ts                          # Express app setup, CORS, middleware
├── index.ts                        # Server entry point
├── middlewares/
│   ├── authenticate.ts             # Vendor/customer auth (session cookie)
│   └── adminAuthenticate.ts        # Cross-backend admin auth (BreezitAdmin cookie)
├── modules/
│   ├── ai/
│   │   ├── aiController.ts         # Agent orchestration, followups, reminders
│   │   ├── orq/orqController.ts    # ORQ.ai integration, generateOrqAiReplyFn
│   │   └── utils/extract-email-data.utils.ts  # Prompt building, timezone conversion
│   ├── chat/phoneMessage/
│   │   ├── phoneMessageController.ts  # SMS webhook endpoints
│   │   └── phoneMessageService.ts     # SMS AI pipeline
│   ├── common/
│   │   ├── gmail/gmailController.ts   # Email webhook, Agent 0, deduplication
│   │   ├── commonController.ts        # aiConfig CRUD, shared utilities
│   │   └── error/
│   │       ├── errorLoggingService.ts      # Error CRUD + stats
│   │       ├── errorLoggingController.ts   # HTTP endpoints
│   │       ├── errorModel.ts               # MongoDB schema
│   │       ├── resubmitService.ts          # Manual resubmission
│   │       └── slackNotificationService.ts # Slack alerts
│   ├── marketplace/calendar/
│   │   ├── calendarController.ts   # Availability checks, 7AM rule
│   │   └── calendarHelpers.ts      # Aggregation, keyword filtering
│   ├── vendor/calendar/
│   │   ├── calendarController.ts   # Calendar sync, Nylas events
│   │   └── calendarModel.ts        # Calendar MongoDB model
│   └── playground/playgroundController.ts  # AI playground testing
├── test/
│   ├── setup.ts                    # Vitest setup
│   ├── fixtures/                   # Test data
│   ├── unit/                       # Unit tests
│   └── integration/                # Integration tests
└── routes/                         # Express route definitions
```

## Admin Module Source Layout

```
admin/src/
├── app.ts                          # Express app setup
├── index.ts                        # Server entry point
├── modules/
│   ├── common/commonController.ts  # Admin aiConfig, shared utils
│   ├── error/                      # Admin error management
│   ├── retell/                     # AI voice (Retell) management
│   ├── bookings/                   # Booking management
│   ├── user/                       # User management
│   ├── payments/                   # Payment/subscription management
│   └── ...                         # 40+ module directories
├── middlewares/                     # Admin auth middleware
└── routes/                         # Express route definitions
```

## Commands

| Action | Auth Module | Admin Module |
|--------|-------------|--------------|
| Start dev | `cd auth && npm start` | `cd admin && npm start` |
| Run all tests | `cd auth && npm test` | `cd admin && npm test` |
| Run specific test | `cd auth && npx vitest run <path>` | `cd admin && npx jest <path>` |
| Run by pattern | `cd auth && npx vitest run --grep "<pattern>"` | `cd admin && npx jest -t "<pattern>"` |
| Build | `cd auth && npm run build` | `cd admin && npm run build` |

## Feature Docs Index

| Feature | Feature File | Key Source Files | Summary |
|---------|-------------|------------------|---------|
| AI Customer Management | `.claude/features/ai-customer-management.md` | `gmailController.ts`, `aiController.ts`, `phoneMessageService.ts` | Multi-agent email/SMS pipeline with lead detection |
| AI Error Monitoring | `.claude/features/ai-error-monitoring.md` | `errorLoggingService.ts`, `slackNotificationService.ts` | Error tracking + CRITICAL Slack notification rule |
| AI Stats & Categories | `.claude/features/ai-stats-and-categories.md` | `errorLoggingService.ts` (`getAiUsageStats`) | Category × flowType matrix, dashboard aggregation |
| 7 AM Rule | `.claude/features/7am-rule.md` | `calendarController.ts`, `calendarHelpers.ts` | Cross-midnight availability with autoBlock fix |
| Multi-Service Calendar | `.claude/features/multi-service-calendar.md` | `commonController.ts`, `calendarHelpers.ts` | Shared calendar with keyword filtering |
| Timezone Conversion | `.claude/features/timezone-conversion.md` | `extract-email-data.utils.ts` | `convertNaiveDateToUTC()` for AI-extracted dates |
| ORQ Metadata | `.claude/features/orq-metadata.md` | `orqController.ts`, all AI call sites | Structured metadata on all ORQ.ai calls |
| Webhook Deduplication | `.claude/features/webhook-deduplication.md` | `gmailController.ts` (~line 938) | Atomic MongoDB upsert prevents duplicate processing |
| Admin Auth Cross-Backend | `.claude/features/admin-auth-cross-backend.md` | `adminAuthenticate.ts` | Dual cookie auth for admin → auth backend |
| Change User Password | `.claude/features/change-user-password.md` | — | bcrypt hash procedure via MongoDB Compass |
| Duplicate SMS Prevention | `.claude/features/duplicate-sms-prevention.md` | `extract-email-data.utils.ts`, `retellController.ts` | Prevents duplicate SMS when deferred social-hours call isn't answered |
| Booking Package Types | `.claude/features/booking-package-types.md` | `create-booking-obj.util.ts`, `extract-email-data.utils.ts` | Package type dispatch (catering, cateringBeverages, venueRental) in booking creation |
| Availability Checking | `.claude/features/availability-checking.md` | `calendarController.ts`, `calendarHelpers.ts` | isServiceAvailable pipeline, naive UTC dates, 7AM rule, keyword filtering |
| Testing Infrastructure | `.claude/features/testing-infrastructure.md` | `auth/src/test/` | Vitest setup, fixtures, 1050+ tests, patterns & anti-patterns |
| Reminder Tour Date Fix | `.claude/features/reminder-tour-date-fix.md` | `extract-email-data.utils.ts`, `aiController.ts` | Fix reminders sending UTC time instead of venue local time |
| Email After Cal.com Booking | `.claude/features/email-after-cal-booking.md` | `aiController.ts`, `calController.ts`, `extract-email-data.utils.ts` | AI email with venue instructions after Cal.com booking |
| Invalid Email SMS Fallback | `.claude/features/invalid-email-sms-fallback.md` | `gmail.utils.ts`, `aiController.ts`, `errorLoggingService.ts`, `resubmitService.ts` | Reject placeholder emails (declined@, none@), fall back to create_sms deployment |
| Notification Center Fallback | `.claude/features/notification-center-fallback.md` | `notification-center.service.ts`, `booking-notifications.util.ts`, `aiConfigTypes.ts` | Unconfigured vendors get all notifications via their own email/phone |
| AI Disabled Tracking | `.claude/features/ai-disabled-tracking.md` | `bookingController.ts` (`switchAi`), `activity-log.service.ts`, `conversation-logs.component.ts` | Timestamp + AiAnalytics entry when AI disabled; visible in activity log & conversation log |
| AI Prompts Migration | `.claude/features/ai-prompts-migration.md` | `auth/src/modules/ai/prompts/`, `admin/src/modules/aiPrompts/` | Extracted prompts to dedicated collections with override system, versioning & admin UI |
| AnswerConnect Noreply Safeguard | `.claude/features/answerconnect-noreply-safeguard.md` | `gmail.utils.ts`, `aiController.ts`, `gmailController.ts` | Fix AnswerConnect email mismatching + noreply email filter + resubmit grantId |
| Platform Proxy Email Matching | `.claude/features/platform-proxy-email-matching.md` | `gmail.utils.ts`, `gmailController.ts`, `aiController.ts`, `extract-email-data.utils.ts` | Extract platform IDs (Mandrill, mailto, replyTo) for follow-up email matching |
| Email After Call Cal.com Fix | `.claude/features/email-after-call-calcom-fix.md` | `blandController.ts`, `extract-email-data.utils.ts` | Fix email_after_call fabricating appointment times by fetching real Cal.com slots |
| Webhook Health Monitor | `.claude/features/webhook-health-monitor.md` | `webhookHealthMonitor.ts`, `cronJobs.ts` | Cron job detecting missed Nylas webhooks via API→dedup cross-reference |
| Old Thread Filtering | `.claude/features/old-thread-filtering.md` | `gmailController.ts` (~line 3106) | Skip old threads unless booking exists (AI already engaged) |
| Wrong Recipient Guard | `.claude/features/wrong-recipient-guard.md` | `gmail.utils.ts`, `gmailController.ts`, `resubmitService.ts` | Last-line-of-defense guard blocking sends to platform service emails |
| AfterInboundSMS Fix & Migrations | `.claude/features/afterInboundSMS-fix-and-migrations.md` | `aiController.ts`, `migrations/index.ts`, `migrationModel.ts`, `cronJobs.ts` | Fix `.lean()` object method call + reusable migration system |
| Migration System | `.claude/features/migration-system.md` | `migrations/index.ts`, `migrations/types.ts`, `migrationModel.ts`, `cronJobs.ts` | File-per-migration runner for one-time data fixes at startup |
| Strip ORQ.ai Prefixes | `.claude/features/strip-orq-ai-prefixes.md` | `extract-email-data.utils.ts`, `phoneMessageService.ts` | Strip `[SMS]`/`[EMAIL]`/`[timestamp]` prefixes from AI-generated text |
| Initial Inquiry Message | `.claude/features/initial-inquiry-message.md` | `aiController.ts`, `extract-email-data.utils.ts`, `conversation-logs.component.ts` | Save initial message ID to show inquiry email in conversation log |
| Partial Booking ID Recovery | `.claude/features/partial-booking-id-recovery.md` | `aiController.ts`, `extract-email-data.ts`, `extract-email-data.utils.ts` | Recover truncated ObjectIds from AI via prefix-match before `findById` |
| Hello Customer & Mailer-Daemon Fix | `.claude/features/hello-customer-mailer-daemon-fix.md` | `extract-email-data.utils.ts`, `gmailController.ts` | Fix getPrompt ignoring extractedData name + mailer-daemon email override |
| No Same-Day Alternatives Guard | `.claude/features/no-same-day-alternatives-guard.md` | `extract-email-data.utils.ts` | Prevent AI fabricating same-day appointment times when Cal.com has no same-day slots |
| SMS After Inbound Call | `.claude/features/sms-after-inbound-call.md` | `extract-email-data.utils.ts`, `retellController.ts`, `blandController.ts`, `phoneMessageService.ts`, `gmailController.ts` | Configurable SMS after inbound calls + email change detection/resend |
| No-Show Follow-Ups | `.claude/features/no-show-followups.md` | `aiController.ts`, `phoneMessageService.ts` | Automated follow-ups for noShow bookings with rolling anchor scheduling |
| Tripleseat Webhook Pipeline | `.claude/features/tripleseat-webhook-pipeline.md` | `tripleseatController.ts`, `gmailController.ts`, `aiController.ts` | Tripleseat CRM lead → full email AI pipeline (Agent 1→2→3/4) |
| Email Subject Fallback | `.claude/features/email-subject-fallback.md` | `extract-email-data.utils.ts`, `aiController.ts` | Fix empty/"Breezit" email subjects when AI omits `\|*(subject)*\|` delimiter |
| Phone Number Validation | `.claude/features/phone-number-validation.md` | `phoneMessageHelpers.ts`, `phoneMessageController.ts`, `extract-email-data.utils.ts` | Validate phones via libphonenumber-js + isolate SMS errors from email pipeline |

### Adding New Feature Docs

Every feature is documented in two places:

1. **Expanded doc** in `docs/<feature-name>.md` — Full context: problem statement, detailed code walkthrough with snippets, data flow diagrams, changelog. This is the source of truth for human readers.
2. **Condensed doc** in `.claude/features/<feature-name>.md` — Concise reference for Claude: overview, key files table, critical logic summaries. References the expanded doc with `> Full docs: docs/<feature-name>.md` at the top.

After creating both files, add a row to the **Feature Docs Index** table above.

### Feature Implementation Checklist

When implementing a new feature or fixing a bug, always complete all five steps:

1. **Code** — Implement the feature or fix.
2. **Logging** — Add structured `console.log` statements at every decision point and data handoff for production observability. Use a consistent prefix (e.g., `---- SMS`, `[featureName]`) so logs can be grepped. Log inputs, computed flags, gate conditions, and outcomes. This is critical — without logging, production issues like the `tour_inquiry_confirmed` bug are impossible to diagnose.
3. **AI Error Monitoring** — Evaluate whether the feature introduces failure points that should be tracked via `ErrorLoggingService.logAiError()` and `SlackNotificationService.notifyError()`. Add AI error monitoring when: the feature adds a new pipeline stage or agent, introduces external service calls that can fail (API, database queries critical to the flow), or has failure modes that would block email/SMS delivery to a customer. Do NOT add error monitoring for: supplementary/fallback mechanisms where failure means graceful degradation, pure utility functions, or non-critical enhancements (e.g., metadata accumulation). See `.claude/features/ai-error-monitoring.md` for categories, statuses, and the critical Slack rule.
4. **Tests** — Update or add test fixtures (`auth/src/test/fixtures/`) and test cases to cover the new behavior, including edge cases.
5. **Docs (MANDATORY)** — You MUST update or create feature documentation after every code change. This is not optional. For every feature or bug fix:
   - Update the relevant **expanded doc** (`docs/<feature-name>.md`) with problem statement, code walkthrough, and changelog entry.
   - Update the relevant **condensed feature doc** (`.claude/features/<feature-name>.md`) with a concise summary of the change.
   - If no relevant doc exists yet, create both files and add a row to the **Feature Docs Index** table above.
   - If the change affects multiple features, update all relevant docs.

## Code Patterns

### Route → Controller → Service

```
routes/*.ts → controller functions → service classes/functions → Mongoose models
```

### Authentication Middleware

- `authenticate` — vendor/customer routes (checks `session` cookie)
- `adminAuthenticate` — routes accessible from admin app (checks both `session` and `BreezitAdmin` cookies)

### Error Handling

Uses `@foxywedding/common` error classes: `BadRequestError`, `NotAuthorizedError`, `NotFoundError`. Express error handler catches and formats.

### AI Pipeline Pattern

1. Create error doc (status: "processing")
2. Run agents sequentially, updating `agentKey`/`agentLabel` on error doc
3. On success: update status to "successful"
4. On failure: update status to "failed", **send Slack notification**

## Critical Rules

1. **Slack Notifications**: Every `logAiError()` with `status: "failed"` MUST be followed by `SlackNotificationService.notifyError()`. See `.claude/features/ai-error-monitoring.md`.

2. **ORQ Metadata**: All `generateOrqAiReplyFn()` calls MUST pass `OrqMetadata`. See `.claude/features/orq-metadata.md`.

3. **Timezone Conversion**: AI-extracted dates are naive — use `convertNaiveDateToUTC()` before storing or comparing. See `.claude/features/timezone-conversion.md`.

## Migrations

Run-once data fixes executed at server startup before cron jobs.

### Adding a Migration

1. Create `auth/src/services/migrations/YYYY-MM-DD_description.ts`:

```typescript
import { MigrationDef } from "./types";

export const migration: MigrationDef = {
  key: "YYYY-MM-DD_description",
  run: async () => {
    // one-time data fix
  },
};
```

2. Register in `auth/src/services/migrations/index.ts` — add import + append to array.

3. Deploy. Runs once, records key in `migrations` MongoDB collection. Skipped on subsequent starts.

### Conventions

- **Key = filename**: `YYYY-MM-DD_kebab-case-description` (must match between file and `key` property)
- **Append only**: New migrations go at the end of the array (order = execution order)
- **Forward-only**: No rollbacks; to undo, write a new migration
- **Logging**: Use `console.log(\`[migration] ...\`)` prefix

4. **Admin Frontend Cross-Check**: When backend changes introduce or modify error types, agent keys (`agentKey`/`agentLabel`), flow types, error categories, API response shapes, or `messageContext` structures, always check the admin frontend (`FoxyVendor/apps/breezit-admin/`) for hardcoded values or UI sections that need updating. Key files to check:
   - `ai-errors/ai-errors.component.ts` — filter dropdowns (`flowTypes`, `statuses`, `categories`, `skipReasons`)
   - `ai-errors/components/error-detail-dialog/` — context display sections, cross-flow trigger descriptions
   - `ai-stats/ai-stats.component.ts` — category labels in stats dashboard

## Testing

### Auth Module (Vitest)

```
auth/
├── vitest.config.ts
├── src/test/
│   ├── setup.ts          # Global test setup
│   ├── fixtures/          # Shared test data
│   ├── unit/              # Fast, isolated tests
│   └── integration/       # Tests with DB/services
```

Run: `cd auth && npx vitest run` (all) or `cd auth && npx vitest run src/test/unit/calendar/sevenAmRule.test.ts` (specific)

### Admin Module (Jest)

Run: `cd admin && npx jest` (all) or `cd admin && npx jest --testPathPattern=<pattern>`
