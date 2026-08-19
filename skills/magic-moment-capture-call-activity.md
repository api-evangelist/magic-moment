---
name: Capture call activity into Playbook
description: Configure a call provider (including MiiTel), obtain a softphone capability token, run conference control, and write call logs into Magic Moment Playbook.
api: openapi/magic-moment-call-integration-openapi.yml
host: https://call.magicmoment.co.jp
operations: [getCallProviderSetting, createCallProviderSetting, updateCallProviderSetting, getMiitelSetting, createMiitelSetting, GetCapabilityToken, AuthCapabilityToken, ConferenceCreate, ChangeConferenceStatus, createCallLog, createCallLogForMiitel, getCallLog]
generated: '2026-08-13'
method: generated
source: openapi/magic-moment-call-integration-openapi.yml (harvested 2026-08-13 from https://call.magicmoment.co.jp/swagger.json)
---

# Capture call activity into Playbook

Playbook's call integration service records phone conversations as structured sales activity. It
supports a generic call provider plus a dedicated MiiTel path.

## Before you start

- **Base host:** `https://call.magicmoment.co.jp`.
- **Auth:** `x-access-token` header on all authenticated operations (13 of the 18 operations declare it).
  The conference callback operations are called by the telephony provider, not by you.
- **No idempotency contract.** `createCallLog` is a plain `POST` — a retry after a timeout can create a
  duplicate log. Read back with `getCallLog` before retrying.
- **Health:** `healthCheck` (`GET /healthcheck`).

## Steps

1. **Inspect existing configuration** — `getCallProviderSetting` (`GET /call-providers`) and, for MiiTel,
   `getMiitelSetting` (`GET /call-providers/miitel`).
2. **Configure the provider** — `createCallProviderSetting` (`POST /call-providers`) or
   `createMiitelSetting` (`POST /call-providers/miitel`); update with `updateCallProviderSetting`
   (`PUT /call-providers/{settingId}`) or `updateMiitelSetting`.
3. **Get a client capability token** — `GetCapabilityToken`
   (`GET /call-providers/{settingId}/capability-token`) returns the token the web softphone uses to
   connect; `AuthCapabilityToken` (`POST .../capability-token`) authenticates the connecting client.
4. **Run the conference** — `ConferenceCreate` (`POST /call-providers/{settingId}/conference`) returns
   the XML the telephony provider executes; `ChangeConferenceStatus`
   (`POST .../conference/status`) holds and un-holds the call; `ConferenceHoldMusic` and
   `ClientReceiveCallXml` serve the provider-side XML callbacks.
5. **Write the call log** — `createCallLog` (`POST /call-log`), or `createCallLogForMiitel`
   (`POST /call-log/miitel`) when the recording originates in MiiTel.
6. **Read it back** — `getCallLog` (`GET /call-log`) to confirm the activity landed before treating the
   call as captured.
