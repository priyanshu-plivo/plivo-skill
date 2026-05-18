# Plivo Cheatsheet — 80% Common Ops

All requests use:
- Auth: `-u "$PLIVO_AUTH_ID:$PLIVO_AUTH_TOKEN"`
- Base: `https://api.plivo.com/v1/Account/$PLIVO_AUTH_ID/`
- Content-Type: `application/json` (for POSTs)

## Messaging

**Send SMS** — Tier 1
```
POST /Message/
{"src":"+1XXX","dst":"+1YYY","text":"hello","url":"https://your.app/dlr"}
```

**Send MMS** — Tier 1
```
POST /Message/
{"src":"+1XXX","dst":"+1YYY","type":"mms","media_urls":["https://..."]}
```

**Send WhatsApp (approved template)** — Tier 1
```
POST /Message/
{"src":"+WABA","dst":"+1YYY","type":"whatsapp",
 "template":{"name":"otp_alert","language":"en_US",
  "components":[{"type":"body","parameters":[{"type":"text","text":"123456"}]}]}}
```

**List messages** — Tier 0
```
GET /Message/?limit=20&offset=0
```

**Get one message** — Tier 0
```
GET /Message/{message_uuid}/
```

## Verify (OTP)

**Start OTP** — Tier 1
```
POST /Verify/Session/
{"recipient":"+1YYY","channel":"sms","app_uuid":"<verify_app_uuid>"}
```
Returns `session_uuid`. Only `recipient` is required; `app_uuid` falls back to account default.

**Validate OTP** — Tier 0 (read-only effective)
```
POST /Verify/Session/{session_uuid}/
{"otp":"123456"}
```
Response `status`: `verified` / `unverified` / `expired`.

## Voice

**Make outbound call** — Tier 1
```
POST /Call/
{"from":"+1XXX","to":"+1YYY","answer_url":"https://your.app/answer.xml","answer_method":"GET"}
```

**Get call** — Tier 0
```
GET /Call/{call_uuid}/
```

**Hang up active call** — Tier 1
```
DELETE /Call/{call_uuid}/
```

**List active calls** — Tier 0
```
GET /Call/?status=live
```

## Numbers

**Search inventory** — Tier 0
```
GET /PhoneNumber/?country_iso=US&type=local&pattern=415&limit=10
```

**Buy number** — Tier 2 (CONFIRM!)
```
POST /PhoneNumber/{number}/
{"app_id":"<app_uuid>","cnam_lookup":"enabled"}
```
`app_id` optional (uses default_number_app). `cnam_lookup` US-only.

**List your numbers** — Tier 0
```
GET /Number/
```

**Release number** — Tier 2 (CONFIRM!)
```
DELETE /Number/{number}/
```

## Account

**Get account info / balance** — Tier 0
```
GET /
```
Look at `cash_credits` for USD balance.

**Get pricing for a country** — Tier 0 (use for cost estimation before Tier 1/2 spends)
```
GET /Pricing/?country_iso=US
```
Returns `message.outbound.rate`, `voice.outbound.rate`, `phone_numbers.{local,tollfree,mobile}.rate`. All STRINGS.

**List subaccounts** — Tier 0
```
GET /Subaccount/
```

## Quick rules

- Phone numbers always E.164 (`+<country><number>`)
- Multiple destinations: separate by `<` (e.g., `+1234<+5678`)
- DLR webhook: set `url` and `method` in send request
- For US SMS: source number must be in a registered 10DLC campaign — see api/10dlc.md
- For India SMS: include `dlt_entity_id` and `dlt_template_id`

If the operation isn't here, Read `api/<resource>.md` for full surface.
