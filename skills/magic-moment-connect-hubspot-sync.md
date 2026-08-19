---
name: Connect HubSpot and start Playbook sync
description: Authorize Magic Moment Playbook against a HubSpot portal, map Playbook records to HubSpot deals, companies and contacts, and turn sync on.
api: openapi/magic-moment-hubspot-integration-openapi.yml
host: https://hubspot.magicmoment.co.jp
operations: [startHubspotAuth, callbackHubspotAuth, listOauthResults, getHubspotUsers, getDealStagesOfHubspot, getHsLeadStatusOfHubspot, getSyncSettingOfHubspot, createSyncSettingOfHubspot, updateSyncFieldMapOfHubspot, saveSyncIDMapOfHubspot, getSyncIDMapOfHubspot, saveEngagementSyncSettingOfHubspot, getSyncable, toggleSyncable, readSyncWithHubspot]
generated: '2026-08-13'
method: generated
source: openapi/magic-moment-hubspot-integration-openapi.yml (harvested 2026-08-13 from https://hubspot.magicmoment.co.jp/swagger.json)
---

# Connect HubSpot and start Playbook sync

The HubSpot integration service mirrors the Salesforce one, against HubSpot's deal, company and contact
objects.

## Before you start

- **Base host:** `https://hubspot.magicmoment.co.jp`.
- **Auth:** `x-access-token` on every call; three operations also accept `x-api-key`. Tokens come from
  the Playbook product — there is no public signup. API contact: `playbook-api@magicmoment.jp`.
- **No idempotency key, no pagination, no rate-limit headers** anywhere in this API. Read state back
  instead of retrying writes.
- **Errors:** `{"code":<int>,"message":"<string>"}`; see `errors/magic-moment-problem-types.yml`.

## Steps

1. **Authorize HubSpot** — `startHubspotAuth` (`GET /oauth2/authorize/`), then `callbackHubspotAuth`
   (`GET /oauth2/callback/`) with `code`, `state`, `redirect_uri`.
2. **Confirm** — `listOauthResults` (`GET /oauth2/results`).
3. **Read the portal's vocabulary** — `getHubspotUsers` (`GET /crm/users`), `getDealStagesOfHubspot`
   (`GET /crm/deal_stages`), `getHsLeadStatusOfHubspot` (`GET /crm/hs_lead_statuses`). Build mappings
   only from values these return.
4. **Create and refine the sync setting** — `createSyncSettingOfHubspot` (`POST /settings`), then
   `updateSyncFieldMapOfHubspot` (`PATCH /settings/{sync_setting_id}`). Read the current state first
   with `getSyncSettingOfHubspot` (`GET /settings`).
5. **Map record identity** — `saveSyncIDMapOfHubspot` (`PUT /mappings`), verify with
   `getSyncIDMapOfHubspot` (`GET /mappings`).
6. **Map engagements** — `saveEngagementSyncSettingOfHubspot` (`PUT /engagement_sync_settings`).
7. **Enable sync** — check `getSyncable` (`GET /sync/syncable`), then `toggleSyncable`
   (`PUT /sync/syncable`).
8. **Pull from HubSpot** — `readSyncWithHubspot` (`POST /sync/read`) reads HubSpot data into Playbook.

## Teardown

`deleteHubspotAuth` (`DELETE /oauth2/authorize/`) unlinks the portal;
`deleteSyncIDMapOfHubspot` and `deleteEngagementSyncSettingOfHubspot` remove the mappings.
