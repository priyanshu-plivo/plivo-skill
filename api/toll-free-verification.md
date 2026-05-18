# Toll-Free Verification

Required for SMS from US toll-free numbers (8XX). Without verification, throughput is throttled.

Docs: https://www.plivo.com/docs/messaging/api/toll-free-verification

## Endpoints

| Method | Path | Purpose |
|---|---|---|
| POST | /TollfreeVerification/ | Submit — Tier 2 |
| GET | /TollfreeVerification/ | List submissions |
| GET | /TollfreeVerification/{id}/ | Get status |
| POST | /TollfreeVerification/{id}/ | Update |
| DELETE | /TollfreeVerification/{id}/ | Delete (only if not submitted) |

## Submit — key params

- `number` — toll-free number
- `business_name`, `business_website`
- `business_contact_first_name`, `business_contact_last_name`, `business_contact_email`, `business_contact_phone`
- `use_case` — short label (e.g., "Order updates")
- `use_case_summary` — detailed description
- `production_message_sample` — example SMS body
- `optin_workflow` — how users consent (text + screenshot URL)
- `optin_workflow_image_urls`
- `volume` — `LOW`, `LOW_VOLUME`, etc.
- `additional_info`

## Status

`SUBMITTED` → `IN_REVIEW` → `APPROVED` / `REJECTED`. Carrier-side approval: 1–3 weeks.

## Throughput

- Unverified: throttled to ~3 SMS/sec
- Verified: full throughput (but still subject to carrier filtering)

## Gotchas

- Rejection usually about unclear opt-in proof — include screenshots of consent UI
- Approval is external (US carriers via Aerialink/etc.) — Plivo can't fast-track
- One submission per number; updates allowed only in certain states
