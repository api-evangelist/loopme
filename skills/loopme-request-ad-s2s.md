---
name: Request a LoopMe ad server-to-server
description: Call the LoopMe S2S ad request endpoint from a third-party ad server, handle the ad / no-ad / error outcomes, and render the returned ad payload correctly.
api: openapi/loopme-ad-serving-api-openapi.yml
operations: [requestAd]
---

# Request a LoopMe ad server-to-server

Use the LoopMe S2S API when your own ad server needs to source a LoopMe ad for a
device. This is a machine-to-machine surface for exchanges, mediation platforms
and publisher ad servers — not a reporting or account-management API.

## Auth
- There is **no token**. The request is identified by the `appId` LoopMe
  provisions to the publisher, sent as a required query parameter.
- Do **not** send `api_auth_token` here; that credential belongs to the
  Reporting API only (see `skills/loopme-pull-reporting.md`).
- Base: `https://loopme.me/api/s2s`.

## Steps

1. Build a `GET` to `requestAd` (`GET /ads`) with every mandatory parameter:
   `appId`, `ua` (device user agent), `uid` (IDFA / Google Advertiser ID /
   cookie ID), `ip`, `clientid` (your server's name, used for debugging on
   LoopMe's side), `dnt` (`1` disables tracking), `bundleid`, `appname`,
   `sdk` (SDK version delivering the ad), and `exchange`.
2. Add the optional optimization parameters when you have them: `app_version`,
   `pubname`, `lon`, `lat`, `reward` (`1` for a rewarded spot), `storeurl`,
   `lng` (2-letter ISO language).
3. URL-encode the user agent and store URL. Send one request per ad opportunity.
4. Branch on the response:
   - **HTTP 200 with a non-empty `ads` array** — an ad was returned. `ads.script`
     carries the ad HTML (`type: MRAIDv1`), including a LoopMe loader script
     and the impression/click beacon URLs.
   - **HTTP 200 with an empty `ads` array** — no fill. This is a normal
     outcome, not an error. Fail over to your next demand source.
   - **HTTP 404 with `{ "status": "error", "errorMessage": "..." }`** — a request
     error. Read `errorMessage`; do not retry blindly.
5. Render the returned `script` payload in an MRAID-capable container and let it
   fire its own beacons. Do not strip or rewrite the beacon URLs — that breaks
   impression counting.

## Macros
Click-through and third-party tracking URLs support LoopMe macros that the ad
server substitutes at serve time (case-sensitive):
`{ad_delivery_token}` (unique ad ID), `{timestamp}` / `{cachebuster}`,
`{device_id}` / `{md5_device_id}` / `{sha1_device_id}`, `{ip_address}`, and the
consent macros `${GDPR}`, `${GDPR_CONSENT_109}` (TCF v2 TC string),
`${ADDTL_CONSENT}` and `${US_PRIVACY}` (CCPA). Always add a cachebuster to
third-party creatives or impression counts will diverge.

## Rules
- Read-only GET; no idempotency contract, and no rate limits are published
  (`rate-limits/loopme-rate-limits.yml`).
- No sandbox or test mode exists. Test on device with the LoopMe X tester apps
  against a real app key (`sandbox/loopme-sandbox.yml`); the `appId d18c430ad2`
  in the docs is an illustration value, not a working credential.
- Sellers must be authorized in `ads.txt` / `app-ads.txt`
  (`loopme.com, <publisher id>, DIRECT, 6c8d5f95897a5a3b`) — LoopMe only buys
  from authorized sellers.

## Reference
- S2S docs: https://wiki.loopme.cool/publishers/integrations/s2s
- Macros: https://wiki.loopme.cool/demand-partners/macros
