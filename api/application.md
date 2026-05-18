# Applications

Apps bind a phone number (or endpoint) to your webhook URLs for handling voice / SMS.

Docs: https://www.plivo.com/docs/account/api/application

## Endpoints

| Method | Path | Purpose |
|---|---|---|
| POST | /Application/ | Create — Tier 1 |
| GET | /Application/ | List |
| GET | /Application/{app_id}/ | Get |
| POST | /Application/{app_id}/ | Update |
| DELETE | /Application/{app_id}/ | Delete — Tier 2 |

## Key params

- `app_name`
- `answer_url`, `answer_method` — XML callback when call connects
- `hangup_url`, `hangup_method` — call ended
- `fallback_answer_url`, `fallback_method` — used if `answer_url` errors
- `message_url`, `message_method` — incoming SMS
- `default_number_app` (bool) — default for any new rented number
- `default_endpoint_app` (bool) — default for new SIP endpoints
- `sub_account` — assign to subaccount
- `log_incoming_message` — false to skip logging inbound SMS body (singular, not plural — Plivo API quirk)

## Usage

Attach an `app_id` to a Number or Endpoint:
```
POST /Number/{number}/  { "app_id": "<app_id>" }
POST /Endpoint/{ep_id}/ { "app_id": "<app_id>" }
```

Then any call/SMS to that resource hits your webhooks.
