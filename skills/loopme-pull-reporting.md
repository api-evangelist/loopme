---
name: Pull LoopMe reporting statistics
description: Retrieve publisher (app/site) or advertiser (campaign) statistics from the LoopMe Reporting API for a date range and granularity.
api: openapi/loopme-reporting-openapi.yml
operations: [getPublisherReport, getAdvertiserReport]
---

# Pull LoopMe reporting statistics

Use the LoopMe Reporting API to retrieve performance statistics for a LoopMe
account. Publishers read app/site stats; advertisers read campaign stats.

## Auth
- Obtain a 16-character `api_auth_token` from the LoopMe dashboard under
  Account/Settings (Reporting API access must be enabled for the account).
- Pass it as the `api_auth_token` query parameter on every request. It is the
  only credential; there is no OAuth or bearer token.

## Steps

1. Decide the audience:
   - Publisher (apps/sites) -> `getPublisherReport` (`GET /reports/apps`).
   - Advertiser (campaigns) -> `getAdvertiserReport` (`GET /reports/campaigns`).
2. Set the required `date_range`: a predefined value (`today`, `yesterday`,
   `days7`, `days30`, `days90`) or a custom `YYYY-MM-DD:YYYY-MM-DD` range.
3. Optionally set `granularity` (`hour`, `day`, `week`, `month`),
   `country_code` (ISO 3166-1 alpha-3, e.g. `USA`), `platform`
   (`ios`/`android`/`website`), and `group_by` (comma-separated dimensions).
4. Narrow with entity filters: publisher `app_key`; advertiser `campaign_id`,
   `line_item_id`, `creative_id`.
5. Send the GET request. Parse the JSON `series[]`, each with a `date`
   (UTC) and a `totals` metric map. Currency values are USD decimals.

## Errors
Errors come back as `{ "errors": { "<code>": "<description>" } }`. Handle:
- `1001` missing/invalid token — re-check `api_auth_token`.
- `2002` bad `date_range` format.
- `4001` / `4002` unknown `app_key` / `campaign_id`.
- `5001` bad `country_code` (must be three-letter).

## Notes
- All requests are read-only GET; no idempotency key is needed.
- Times are UTC; there is no documented rate limit.
