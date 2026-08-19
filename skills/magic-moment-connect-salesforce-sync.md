---
name: Connect Salesforce and start Playbook sync
description: Authorize Magic Moment Playbook against a Salesforce org, map Playbook objects and fields to Salesforce objects and fields, and turn bidirectional sync on.
api: openapi/magic-moment-salesforce-integration-openapi.yml
host: https://sfdc.magicmoment.co.jp
operations: [startSfdcAuth, callbackSfdcAuth, listOauthResults, getSalesforceUsers, getOpportunityStageOfSfdc, createSyncSettingOfSfdc, updateSyncFieldMapOfSfdc, saveSyncIDMapOfSfdc, getSyncIDMapOfSfdc, saveEngagementSyncSettingOfSfdc, getSyncable, toggleSyncable, readSyncWithSfdc]
generated: '2026-08-13'
method: generated
source: openapi/magic-moment-salesforce-integration-openapi.yml (harvested 2026-08-13 from https://sfdc.magicmoment.co.jp/swagger.json)
---

# Connect Salesforce and start Playbook sync

Magic Moment Playbook writes captured sales activity back into Salesforce. This skill covers the
connection lifecycle for the Salesforce integration service.

## Before you start

- **Base host:** `https://sfdc.magicmoment.co.jp` (the served contract omits `host`/`basePath`; this
  host is where the contract is published and is listed in the magicmoment.co.jp TLS certificate).
- **Auth:** send `x-access-token: <playbook access token>` on every call. Four operations also accept
  `x-api-key`. Tokens are issued through the Playbook product; there is no public self-serve signup, and
  the developer portal at `https://developer.magicmoment.co.jp` sits behind the product login. API
  access questions go to `playbook-api@magicmoment.jp`.
- **No idempotency contract.** No `Idempotency-Key` header exists anywhere in this API. Do not blindly
  retry `POST`/`PUT`/`PATCH` — read state back first (`getSyncSettingOfSfdc`, `getSyncIDMapOfSfdc`) and
  reconcile.
- **No pagination and no rate-limit headers.** Collection reads are filtered by query parameters, not
  paged. Nothing tells you your budget; back off on your own schedule.
- **Errors** come back as `{"code":<int>,"message":"<string>"}` from this service and as
  `{"head":{"success":false,"code":<int>,"message":"..."},"body":null}` from the platform gateway. See
  `errors/magic-moment-problem-types.yml`.

## Steps

1. **Start the Salesforce authorization** — `startSfdcAuth` (`GET /oauth2/authorize/`). This is Magic
   Moment acting as an OAuth 2.0 client of Salesforce; redirect the human through the returned flow.
2. **Handle the callback** — `callbackSfdcAuth` (`GET /oauth2/callback/`) receives `code`, `state` and
   `redirect_uri` and completes the link.
3. **Confirm the link** — `listOauthResults` (`GET /oauth2/results`) lists authorization results for the
   tenant. Do not continue until the Salesforce result is present.
4. **Read the target org's vocabulary** — `getSalesforceUsers` (`GET /crm/users`) and
   `getOpportunityStageOfSfdc` (`GET /crm/opportunity_stages`) so the mapping you build uses real
   Salesforce user ids and stage values rather than guesses.
5. **Create the sync setting** — `createSyncSettingOfSfdc` (`POST /settings`), then refine field
   mapping with `updateSyncFieldMapOfSfdc` (`PATCH /settings/{sync_setting_id}`).
6. **Map record identity** — `saveSyncIDMapOfSfdc` (`PUT /mappings`) establishes the Playbook ↔
   Salesforce id correspondence; verify with `getSyncIDMapOfSfdc` (`GET /mappings`).
7. **Map engagements** — `saveEngagementSyncSettingOfSfdc` (`PUT /engagement_sync_settings`) controls how
   Playbook engagements land on Salesforce activity/opportunity records.
8. **Check, then enable sync** — `getSyncable` (`GET /sync/syncable`) reports current sync state;
   `toggleSyncable` (`PUT /sync/syncable`) starts or stops it.
9. **Pull from Salesforce on demand** — `readSyncWithSfdc` (`POST /sync/read`) reads Salesforce data into
   Playbook. Treat this as expensive and non-idempotent; check `getSyncable` first.

## Teardown

`deleteSfdcAuth` (`DELETE /oauth2/authorize/`) unlinks the org. Mappings and sync settings are removed
separately with `deleteSyncIDMapOfSfdc` and `deleteEngagementSyncSettingOfSfdc`.
