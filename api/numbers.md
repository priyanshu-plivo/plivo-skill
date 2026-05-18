# Phone Numbers

Search inventory, rent, list, configure, release.

Docs: https://www.plivo.com/docs/numbers/phone-numbers

## Search & Rent

| Method | Path | Purpose |
|---|---|---|
| GET | /PhoneNumber/ | Search available inventory |
| POST | /PhoneNumber/{number}/ | Rent (Tier 2 — recurring cost) |

Search params:
- `country_iso` — REQUIRED. ISO 3166 alpha-2 (e.g. `US`, `IN`, `GB`)
- `type` — `local`, `mobile`, `tollfree`, `national`, `fixed`
- `pattern` — prefix match (e.g. `415`)
- `npanxx` — 6-digit prefix (US/Canada only)
- `local_calling_area` — bool, expand search
- `region` — for `fixed` type
- `city` — for `local` type
- `services` — csv: `voice`, `sms`, `mms`, `voice,sms`, `voice,sms,mms`
- `lata`, `rate_center` — US/Canada only
- `limit` — max 20 (default 20), `offset` — pagination

Rent body (optional):
```json
{"app_id":"<app_uuid>","cnam_lookup":"enabled"}
```
- `app_id` — defaults to `default_number_app` if omitted
- `cnam_lookup` (US only) — `enabled` or `disabled`

Buy response (HTTP 201):
```json
{"status":"fulfilled","numbers":[{"number":"17735723295","status":"Success"}],"message":"created","api_id":"..."}
```
Note: uses `numbers[]`, not `objects[]`. Status is top-level + per-number.

## GET /Number/{number}/ response — important fields

- `active`, `added_on`, `renewal_date` — lifecycle
- `application` — URI string `/v1/Account/{auth_id}/Application/{app_id}/`, NOT bare ID (parse to extract app_id)
- `carrier`, `country`, `city`, `region`
- `monthly_rental_rate`, `voice_rate`, `sms_rate`, `mms_rate` — all STRINGS
- `voice_enabled`, `sms_enabled`, `mms_enabled` — bools
- `tendlc_campaign_id`, `tendlc_registration_status` — US 10DLC state (`UNREGISTERED` until linked)
- `compliance_application_id`, `compliance_status` — link to compliance app
- `cnam`, `cnam_lookup`, `cnam_registration_status` — CNAM (US)
- `toll_free_sms_verification*` — null for non-toll-free
- `verification_info` — array, caller-ID verification state
- `sub_account` — null if main-account-owned

POST /Number/{number}/ update returns HTTP **202 "changed"** (async accept), not 200.

## Manage rented numbers

| Method | Path | Purpose |
|---|---|---|
| GET | /Number/ | List your numbers |
| GET | /Number/{number}/ | Get details |
| POST | /Number/{number}/ | Update (app, subaccount assignment) |
| DELETE | /Number/{number}/ | Release (Tier 2 — irreversible) |

Update body:
```json
{"app_id":"<new_app_uuid>","subaccount":"<subauth_id>"}
```

## Cost

- Rent: monthly recurring (varies $0.50–$5+/month by country/type)
- Per-call / per-SMS charges separate
- Release: free but may not be re-rentable (returns to carrier pool)

## Gotchas

- Renting reserves the number immediately; auto-renews monthly
- US toll-free: requires toll-free verification before SMS-capable (api/toll-free-verification.md)
- US local: requires 10DLC campaign link before SMS to consumers (api/10dlc.md)
- Some countries require compliance docs (api/compliance.md)
- Releasing during a billing cycle: no proration

## Not available via API

- Number porting (manual console)
- CNAM registration (manual console)
- See not-supported.md
