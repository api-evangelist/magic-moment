---
name: Sync calendar and mail into Playbook
description: Connect Google Workspace or Microsoft 365, read and write calendar events, and drive mail ingestion into Magic Moment Playbook through the office-suite integration service.
api: openapi/magic-moment-office-suite-integration-openapi.yml
host: https://suite.magicmoment.co.jp
operations: [startAuth, callbackAuth, listOauthResults, deauthorization, listCalendarEvents, listUserCalendarEvents, createCalendarEvent, showCalendarEvents, updateCalendarEvent, deleteCalendarEvent, enqueueOauthResultsForSyncMail, syncMailByOauthResultId, syncMailByImap, getMailAttachment, updateMicrosoft365SubscriptionExpiration, getProfile]
generated: '2026-08-13'
method: generated
source: openapi/magic-moment-office-suite-integration-openapi.yml (harvested 2026-08-13 from https://suite.magicmoment.co.jp/swagger.json)
---

# Sync calendar and mail into Playbook

The office-suite (Office suite service連携) service is how Playbook captures meetings and email from
Google Workspace and Microsoft 365 — and where Zoom Phone call logs land.

## Before you start

- **Base host:** `https://suite.magicmoment.co.jp`.
- **Auth:** `x-access-token` on 17 operations; four take `x-api-key` (declared as the `apiToken` scheme
  in this contract). Five operations declare an explicit `401` response — this is the only one of the
  five Playbook contracts that documents its auth failure in-spec.
- **One deprecated operation:** `syncMails` (`PATCH /mail/sync`) is marked `deprecated: true`; the
  contract says client-polling mail sync is being retired. Use the oauth-result driven path below
  instead.
- **No idempotency contract.** `createCalendarEvent` is a plain `POST`; verify with `listCalendarEvents`
  before retrying.

## Steps

### Connect the mailbox / calendar

1. `startAuth` (`GET /oauth2/authorize`) with `service_type` for the target suite, then `callbackAuth`
   (`GET /oauth2/callback`).
2. Confirm with `listOauthResults` (`GET /oauth2/results`). `deauthorization`
   (`POST /oauth2/deauthorization`) and `deleteAuth` (`DELETE /oauth2/authorize`) unlink it.
3. For Microsoft 365, keep the change-notification subscription alive with
   `updateMicrosoft365SubscriptionExpiration` (`PATCH /microsoft365/subscription-expiration`). Microsoft
   posts notifications back to `microsoft365WebhookSubscription`
   (`POST /microsoft365/webhook/subscription`) — an inbound receiver, not a webhook you can subscribe to.

### Calendar

4. `listCalendarEvents` (`GET /calendars/events`) or `listUserCalendarEvents`
   (`GET /calendars/{userId}/events`) to read; `showCalendarEvents`
   (`GET /calendars/{calendarId}/{eventId}`) for one event.
5. `createCalendarEvent` (`POST /calendars`), `updateCalendarEvent`
   (`PUT /calendars/{calendarId}/{eventId}`), `deleteCalendarEvent` (`DELETE` on the same path).

### Mail

6. `enqueueOauthResultsForSyncMail` (`PATCH /mail/oauth-results/enqueue`) queues the connected accounts
   for ingestion.
7. `syncMailByOauthResultId` (`PATCH /mail/oauth-results/{oauthResultId}/sync`) ingests one account's
   mail; `syncMailByImap` (`PATCH /mail/smtp-imap-credentials/sync`) does the same for IMAP/SMTP
   credentials.
8. `getMailAttachment` (`GET /mail/attachment/{mailAttachmentId}`) downloads an attachment. This
   response declares `Accept-Ranges`, `Content-Length` and `Content-Range` — it supports ranged reads.

### Zoom Phone

9. `createZoomPhoneCallLog` (`POST /zoom-phone/call-logs`), `saveUserRecording`
   (`POST /zoom-phone/call-logs/recording`) and `getZoomPhoneRecodingDataStream`
   (`GET /zoom-phone/call-logs/recordings/{recodingId}/stream`) persist and stream Zoom Phone calls;
   `verifyZoomPhoneActive` (`GET /zoom-phone/{userId}/status`) checks the link first.
