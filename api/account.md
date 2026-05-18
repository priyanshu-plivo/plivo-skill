# Account

Read-only account info, balance, profile updates.

Docs: https://www.plivo.com/docs/account/api/account

## Endpoints

| Method | Path | Purpose |
|---|---|---|
| GET | / | Get account info |
| POST | / | Update profile (name, address, city) |

## Response fields

- `auth_id`
- `name`
- `cash_credits` — current balance, returned as STRING (e.g. `"10"` not `10`). Parse before doing math. No currency field; convention is USD per the Plivo Console.
- `auto_recharge` — bool
- `billing_mode` — `prepaid` or `postpaid`
- `address`, `city`, `state`, `timezone`
- `account_type` — `standard` / `enterprise`
- `api_id`
- `resource_uri` — self-link (Plivo returns this on every resource response)

## Update

```json
{"name":"Company Inc","city":"Austin","address":"123 Main St"}
```

## Not exposed via API

- Add funds / recharge (wire transfer, manual)
- Payment method (cards)
- Plan tier change
- API key / Auth Token rotation
- 2FA / login security
- Team / user management

See `not-supported.md` for the full list.
