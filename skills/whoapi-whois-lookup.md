---
name: Look up WHOIS records with WhoAPI
description: Retrieve WHOIS registration data, domain age, and reputation signals for a domain using WhoAPI.
api: openapi/whoapi-openapi.yml
operations: [queryWhoAPI]
---

# WHOIS lookup

Use this skill to fetch WHOIS records and registration data for a domain.

## Auth
Pass your private key as the `apikey` query parameter.

## Steps
1. Call `queryWhoAPI` (GET `https://api.whoapi.com/`) with:
   - `r=whois`
   - `domain=<the domain including TLD>`
   - `apikey=<your key>`
2. Parse the JSON response; check `status` (0 = success). On a non-zero status consult `errors/whoapi-error-codes.yml` — e.g. `7` = WHOIS server not yet supported, `30` = WHOIS server problem (retry later), `31` = node WHOIS limit reached (make a new request).
3. To also assess reputation, follow up with `r=domainscore`; to inspect the certificate, `r=cert`; to check blacklists, `r=blacklist` (add `ip=`).

## Example
`https://api.whoapi.com/?domain=whoapi.com&r=whois&apikey=YOUR_API_KEY`

## Notes
- Responses are gzip-compressed JSON (`application/json; charset=utf-8`).
- No pagination or idempotency-key contract; each call is a standalone safe GET read.
