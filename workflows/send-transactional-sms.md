# Workflow: Send Transactional SMS

Country specifics matter. US needs 10DLC. India needs DLT. Elsewhere usually direct.

## US (10DLC-compliant)

Pre-req: number is linked to a registered, VERIFIED 10DLC campaign (api/10dlc.md).

```bash
curl -X POST "https://api.plivo.com/v1/Account/$PLIVO_AUTH_ID/Message/" \
  -u "$PLIVO_AUTH_ID:$PLIVO_AUTH_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "src":"+1<your_10dlc_number>",
    "dst":"+1<recipient>",
    "text":"Your order shipped. Track: https://example.com/t/abc",
    "url":"https://your.app/sms-dlr",
    "log":false
  }'
```

`log:false` redacts body from Plivo logs (PII-sensitive content).

## India (DLT-compliant)

Pre-req: DLT entity + template registered on the Indian DLT portal.

```json
{
  "src":"<SENDERID>",
  "dst":"+91XXXXXXXXXX",
  "text":"Your OTP is 123456. Valid for 5 mins. -ACME",
  "dlt_entity_id":"<entity_id>",
  "dlt_template_id":"<template_id>"
}
```

## Status callback (DLR)

Plivo POSTs to your `url` with:
- `MessageUUID`
- `Status` — `queued`, `sent`, `delivered`, `undelivered`, `failed`
- `ErrorCode` — Plivo or carrier code
- `To`, `From`
- `Units` — segment count
- `TotalRate`, `TotalAmount`

## Common errors

- 117 (US) — source not in registered 10DLC campaign
- 102 — invalid destination format (not E.164)
- 120 — message too long after segmentation
- 4720, 4730 — US carrier filtering (typically unregistered or flagged content)

## Cost watch

Charge is per-segment. Long messages with emoji = UCS-2 = 70 chars/segment instead of 160. A 200-char marketing message with emoji = 3 segments = 3x cost.
