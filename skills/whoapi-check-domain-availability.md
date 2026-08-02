---
name: Check domain availability with WhoAPI
description: Determine whether a domain name is registered (taken) or available using the WhoAPI availability task.
api: openapi/whoapi-openapi.yml
operations: [queryWhoAPI]
---

# Check domain availability

Use this skill to check if a domain is registered using WhoAPI.

## Auth
Pass your private key as the `apikey` query parameter. There is no OAuth; a single API key authenticates every request.

## Steps
1. Call `queryWhoAPI` (GET `https://api.whoapi.com/`) with:
   - `r=taken`
   - `domain=<the domain including TLD>` (e.g. `example.com`)
   - `apikey=<your key>`
2. Parse the JSON response.
3. **Always branch on the numeric `status` field first**, not the HTTP status (errors return HTTP 200):
   - `status: 0` — success; read the availability result fields.
   - non-zero — look up the code in `errors/whoapi-error-codes.yml` (e.g. `4` = TLD does not exist, `17`/`36` = query limit exceeded, `10` = API key invalid).

## Example
`https://api.whoapi.com/?domain=whoapi.com&r=taken&apikey=demokey`

## Notes
- The `demokey` demo key (see `sandbox/whoapi-sandbox.yml`) can be used for trial requests.
- Availability is checked across hundreds of TLDs; see the supported-TLD list.
