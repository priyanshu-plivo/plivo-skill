# Verify (OTP)

OTP generation, delivery, validation. Channels: SMS, voice.

Docs: https://www.plivo.com/docs/programmable-api/verify/sessions

## Endpoints

| Method | Path | Purpose |
|---|---|---|
| POST | /Verify/Session/ | Start OTP session |
| POST | /Verify/Session/{session_uuid}/ | Validate OTP |
| GET | /Verify/Session/{session_uuid}/ | Get session |
| GET | /Verify/Session/ | List sessions (last 90 days; rate-limit 20/min) |

## Start session

Required:
- `recipient` (E.164)
- `app_uuid` — Verify App UUID. **Plivo docs say this is optional with account default, but in practice the API returns HTTP 400 `"'' is not a valid app_uuid"` if omitted without a default Verify app configured.** Always pass it (or create a default Verify App in Console first).

Optional:
- `channel` — `sms` (default) or `voice`
- `url`, `method` — status callback
- `code_length` — 4–10 digits
- `locale` — message language
- `brand_name` — prefix in SMS body (e.g. "Acme: Your code is 123456")
- `app_hash` — Android SMS Retriever API hash for auto-fill

```json
{
  "recipient":"+1234567890",
  "app_uuid":"<verify_app_uuid>",
  "channel":"sms",
  "code_length":6,
  "url":"https://your.app/verify-dlr"
}
```

Response: `session_uuid` + status `in-progress`.

## Validate

```json
{"otp":"123456"}
```

Response `status`: `verified`, `unverified` (retry allowed), `expired`, `failed`.

## Filters (GET list)

`status`, `recipient`, `app_uuid`, `time_init__gte`, `time_init__lt`, `limit`, `offset`.

## Gotchas

- Max 10 validation attempts per session
- Default OTP TTL: 600 seconds (10 min)
- Voice channel reads code aloud — counts as voice minutes
- Brute-force protection: lockout after repeated failures from same IP
- Only one active session per recipient at a time
- Voice OTP needs a verified caller ID or owned Plivo number
- `GET /Verify/Session/` (list) returns HTTP 400 `"no attempts found"` if zero sessions exist on the account — treat 400 as empty-state, not error
