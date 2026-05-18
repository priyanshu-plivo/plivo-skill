# SIP Endpoints

SIP user accounts (softphones, IP phones) registered with Plivo.

Docs: https://www.plivo.com/docs/voice/api/endpoints

## Endpoints (heh)

| Method | Path | Purpose |
|---|---|---|
| POST | /Endpoint/ | Create — Tier 1 |
| GET | /Endpoint/ | List |
| GET | /Endpoint/{ep_id}/ | Get |
| POST | /Endpoint/{ep_id}/ | Update |
| DELETE | /Endpoint/{ep_id}/ | Delete |

## Create params

- `username` — SIP username (will be `<username>@phone.plivo.com`)
- `password`
- `alias` — display name
- `app_id` — XML app to handle incoming calls to this endpoint

## SIP URI format

`sip:<username>@phone.plivo.com`

## Use case

Soft phones / mobile SDK clients that need to register with Plivo's SIP infrastructure for inbound calls or to originate outbound calls without a PSTN number.
