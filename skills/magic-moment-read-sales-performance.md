---
name: Read sales performance out of Playbook
description: Pull team, rep and playbook-phase performance summaries, achievement against targets, and MRR/contract sales reports out of the Magic Moment Playbook reporting service.
api: openapi/magic-moment-reporting-openapi.yml
host: https://report.magicmoment.co.jp
operations: [getTeamPerformanceSummary, getRepPerformanceSummary, getPhasePerformanceSummary, getTeamPerformanceDetail, getRepPerformanceDetail, getPhasePerformanceDetail, getEngagementIdsOfPerformanceSummary, getUnitAchievement, getUserAchievement, getUnitPerformanceTarget, getUserPerformanceTarget, getMrrSalesReport, getContractSalesReport, getPlaybookSummaryReport, getPlaybookDetailReport]
generated: '2026-08-13'
method: generated
source: openapi/magic-moment-reporting-openapi.yml (harvested 2026-08-13 from https://report.magicmoment.co.jp/swagger.json)
---

# Read sales performance out of Playbook

The reporting (集計基盤) service is the read side of Playbook: what the captured activity adds up to.

## Before you start

- **Base host:** `https://report.magicmoment.co.jp`.
- **Auth:** `x-access-token` on 20 of the 22 operations; one accepts `x-api-key`.
- **Filtering, not paging.** These reads take `startDate`/`endDate`, `team_id`, `ownerId`,
  `contractedUnitIds[]`, `contractedUserIds[]`, `productIds[]` and similar. There is no cursor or page
  parameter — narrow with filters instead.
- **Writes are aggregation jobs.** `savePlaybookReport` (`PUT /aggregate/playbook-phases`) and
  `saveDailyPlaybookReport` (`PUT /aggregate/playbook-phases/daily`) rebuild aggregates; the daily one
  can answer `409 Conflict` when an aggregation is already running. Treat a 409 as "already in flight",
  not as a failure to retry immediately.

## Steps

1. **Start at the summary level** — `getTeamPerformanceSummary`
   (`GET /reports/performance-summary/team`), `getRepPerformanceSummary`
   (`GET /reports/performance-summary/rep`), `getPhasePerformanceSummary`
   (`GET /reports/performance-summary/phase`).
2. **Drill into detail** — `getTeamPerformanceDetail`, `getRepPerformanceDetail`
   (`GET /reports/performance-detail/rep/{engagementOwnerId}`) and `getPhasePerformanceDetail`
   (`GET /reports/performance-detail/phase/{phaseValue}`).
3. **Resolve the underlying deals** — `getEngagementIdsOfPerformanceSummary`
   (`GET /reports/engagement-ids`) returns the engagement ids behind a summary, so a number can be traced
   to the deals that produced it.
4. **Compare against targets** — `getUnitPerformanceTarget` and `getUserPerformanceTarget`, then
   `getUnitAchievement` (`GET /reports/achievement/unit`) and `getUserAchievement`
   (`GET /reports/achievement/user`).
5. **Read the money** — `getMrrSalesReport` (`GET /reports/sales/mrr`) and `getContractSalesReport`
   (`GET /reports/sales/contract`).
6. **Read playbook execution** — `getPlaybookSummaryReport`
   (`GET /reports/playbook-phases/{playbookPhaseId}/summary`), `getPlaybookDetailReport`
   (`.../detail`), `getPlaybookItemSummaryReport` (`.../items/summary`) and `listPlaybookItemWordCounts`
   (`.../items/{playbookItemId}/word-counts`).
