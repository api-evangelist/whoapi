---
name: Score a domain or email asynchronously with WhoAPI
description: Run the two-step WhoAPI reputation scoring flow (set task, then receive the result by webhook or poll the check task) for domain reputation and email reputation.
api: openapi/whoapi-domain-intelligence-api-openapi.yml
operations: [queryWhoAPI]
---

# Score a domain or email asynchronously

WhoAPI's reputation products are the only tasks that are **not** real-time. Scoring runs
multiple checks server-side, so it is a two-step flow: submit the subject, then collect the
result. Calling the check task without first submitting returns an empty reply.

## Auth
Pass your private key as the `apikey` query parameter. One API key per account; generating a
new key immediately invalidates the old one. Optional IP whitelisting is configured in the
account settings — a request from a non-whitelisted IP returns `status: 15`.

## Step 1 — submit for scoring (set task)

Call `queryWhoAPI` (GET `https://api.whoapi.com/`) with:

- Domain reputation: `r=domainscore` and `domain=<domain including TLD>`
- Email reputation: `r=emailscore` and `email=<address>`
- `apikey=<your key>`
- optional: `webhook_url=<https URL you control>`

Supplying `webhook_url` is the difference between a push and a poll. It is set **per request**
— there is no webhook registration or management API.

## Step 2a — receive the callback (preferred)

WhoAPI POSTs `application/json` to your `webhook_url` when scoring finishes. The body is an
**array** of result objects whose fields match the check-task reply:

```json
[{"id":9673,"domain":"whoapi.com","overall_score":68,"temp_email_service":false,"score_description":"Good"}]
```

Handle it as an array, not a single object. On any HTTP error from your endpoint WhoAPI retries
**5 times** with a fixed ladder of **5, 15, 30, 60, 60 minutes**, then gives up.

> **No signature.** WhoAPI documents no HMAC header, shared secret, or verification procedure
> for these callbacks. Treat the payload as unauthenticated: use an unguessable callback path,
> and re-verify anything that drives a decision by calling the check task in Step 2b.

## Step 2b — poll instead (fallback)

Call `queryWhoAPI` with `r=domainscore-check` (or `r=emailscore-check`) and the same
`domain`/`email`. A `0` reply means the result is not ready — or that Step 1 was skipped. Back
off before retrying; each poll consumes a request from the monthly allowance.

## Error handling

**Always branch on the numeric `status` field, not the HTTP status** — every error is returned
inside an HTTP 200 body.

- `0` — success
- `10` / `12` — API key invalid / invalid API account
- `15` — IP not whitelisted
- `16` — burst throttle; your IP is blocked for one minute. Wait 60s before retrying.
- `17` / `18` / `36` — plan, high-usage, or monthly quota exhausted. **Do not retry** — the
  quota resets with the billing period. See `rate-limits/whoapi-rate-limits.yml`.
- `33` / `34` / `35` — free-tier exhausted; the account must upgrade.

The full 43-code catalog is in `errors/whoapi-error-codes.yml`. There is no `Retry-After`
header and no `429`, so the retry decision must come from the `status` value alone.

## Notes

- Domain Score and Email Score are billed as separate subscriptions
  (`plans/whoapi-plans-pricing.yml`); "Webhook support" is listed on every paid tier of both.
- The `webhook_url` parameter is missing from WhoAPI's main request-parameter table; it is
  documented only in the per-product guides and captured in
  `overlays/whoapi-openapi-overlay.yaml`.
- Callback catalog and retry policy: `asyncapi/whoapi-webhooks.yml`.
