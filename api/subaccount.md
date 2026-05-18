# Subaccount

Isolated child accounts under your main account. Multi-tenant primitive.

Docs: https://www.plivo.com/docs/account/api/subaccount

## Endpoints

| Method | Path | Purpose |
|---|---|---|
| POST | /Subaccount/ | Create — Tier 1 (HTTP 201) |
| GET | /Subaccount/ | List |
| GET | /Subaccount/{subauth_id}/ | Get |
| POST | /Subaccount/{subauth_id}/ | Update (rename, enable/disable) — Tier 1 (HTTP 202) |
| DELETE | /Subaccount/{subauth_id}/ | Delete — Tier 2 (HTTP 204, permanent) |

## Create

```json
{"name":"tenant-acme","enabled":true}
```
Response (HTTP 201):
```json
{
  "auth_id": "SAXXXXXXXXXXXXXXXXXX",
  "auth_token": "<sub_token>",
  "message": "created",
  "api_id": "..."
}
```
SAVE the `auth_token` — you cannot retrieve it later (only `auth_id` is queryable; token visible in GET response too but rotation is Console-only).

## Get / Read

GET fields:
- `account` — parent URI `/v1/Account/{main_auth_id}/`
- `auth_id`, `auth_token`, `new_auth_token` — credentials (token rotation state)
- `name`, `enabled`
- `created`, `modified` — ISO dates
- `resource_uri` — self link

## Update

POST returns HTTP **202 "changed"** (async accept).

```json
{"name":"renamed"}
```
```json
{"enabled":false}
```
Disabling: subaccount API calls fail until re-enabled; existing resources frozen but not deleted. Reversible.

## Delete

HTTP **204** (no content). Permanent.

## Operating as a subaccount

Two equivalent ways:

**Method 1 — subaccount's own creds:**
```bash
curl -u "$SUB_AUTH_ID:$SUB_AUTH_TOKEN" \
  https://api.plivo.com/v1/Account/$SUB_AUTH_ID/Message/
```

**Method 2 — main account scoping subaccount in URL:**
```bash
curl -u "$MAIN_AUTH_ID:$MAIN_AUTH_TOKEN" \
  https://api.plivo.com/v1/Account/$SUB_AUTH_ID/Message/
```
Method 2 works because main account has parent access.

## Transferring resources between main and subaccount

**Numbers** — move via POST /Number/{number}/ update:
```bash
POST /v1/Account/$MAIN_AUTH_ID/Number/{number}/
{"subaccount": "$SUB_AUTH_ID"}
```
To move back to main: `{"subaccount": null}` or empty string.

**Applications** — assign via POST /Application/{app_id}/:
```bash
POST /v1/Account/$MAIN_AUTH_ID/Application/{app_id}/
{"sub_account": "$SUB_AUTH_ID"}
```
**FIELD NAME WARNING**: `sub_account` (with underscore) on Application, but `subaccount` (no underscore) on Number. Plivo inconsistency.

**Endpoints** — same pattern via /Endpoint/{ep_id}/ update.

## Listing a subaccount's resources from main

Filter by `sub_account` on most list endpoints:
```bash
GET /v1/Account/$MAIN_AUTH_ID/Message/?sub_account=$SUB_AUTH_ID
GET /v1/Account/$MAIN_AUTH_ID/Call/?sub_account=$SUB_AUTH_ID
GET /v1/Account/$MAIN_AUTH_ID/Recording/?sub_account=$SUB_AUTH_ID
```

## Billing & limits

- Charges hit MAIN account always — subaccount has no separate balance or budget
- No per-subaccount credit limit, monthly cap, or spend ceiling via API (see not-supported.md)
- For cost reporting, aggregate CDRs by `sub_account` field

## Auth tokens

- Subaccount auth_token is independent — rotating sub doesn't affect main
- Token rotation is Console-only (no API — see not-supported.md)
- `new_auth_token` on GET response: typically equals `auth_token`; differs only during a rotation flow

## Deletion side effects

- Numbers: transferred back to main account (NOT released — still incurring monthly charge)
- Applications, endpoints, SIP creds belonging to sub: deleted with it
- CDRs (past messages/calls): remain accessible via main, filtered by `sub_account`
- Recordings: remain on Plivo CDN
- Cannot be undone or recovered

## Gotchas

- Field name inconsistency: `subaccount` (Number) vs `sub_account` (Application/list filters)
- POST update returns 202, not 200 — don't fail on non-200
- Delete is immediate and permanent; consider disabling first as soft-pause
- Tenant credentials = security boundary; rotate via Console if exposed
- Charges still hit main — your billing system must track per-tenant usage
