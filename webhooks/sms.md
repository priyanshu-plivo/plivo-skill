# SMS Webhooks

## Delivery Report (DLR)

Plivo POSTs to the `url` you passed in /Message/ POST (or app-level `message_url`).

Triggered: every status transition.

Form-encoded params:
- `From` — source number/sender ID
- `To` — destination
- `MessageUUID` — Plivo's UUID
- `Status` — `queued`, `sent`, `delivered`, `undelivered`, `failed`
- `ParentMessageUUID` — for multi-segment, the parent
- `PartInfo` — segment info (e.g. `1 of 3`)
- `TotalRate` — cost per segment (STRING)
- `TotalAmount` — total charge (STRING)
- `Units` — segment count
- `ErrorCode` — Plivo or carrier code (when undelivered/failed)
- `MCC`, `MNC` — mobile country/network code (if available)

Example handler input (POST body):
```
MessageUUID=abc-def-123&Status=delivered&From=15551234567&To=15559876543&Units=1&TotalRate=0.0077&TotalAmount=0.0077
```

## Common ErrorCode values

- `117` — US, source not in registered 10DLC campaign
- `102` — invalid destination format
- `120` — message too long after segmentation
- `4720`, `4730`, `4750`, `4770` — US carrier filtering (spam-flagged)
- `132000` — WhatsApp template not found / parameter mismatch
- `131047` — WhatsApp 24h window expired
- `131056` — invalid WhatsApp recipient

## Inbound SMS

When someone texts your Plivo number, Plivo POSTs to the app's `message_url`.

Form-encoded params:
- `From` — sender
- `To` — your Plivo number
- `Text` — message body
- `MessageUUID`
- `Type` — `sms` or `mms`
- `MessageIntent` — `transactional` / `promotional` (US-only)
- `MediaUrl0`, `MediaUrl1`, ... — MMS media URLs
- `MediaContentType0`, ... — MIME types
- `TotalRate`, `TotalAmount`, `Units`

Return 2xx. Optionally return Plivo XML to auto-reply:
```xml
<Response>
  <Message src="$To" dst="$From">Thanks, we got your message.</Message>
</Response>
```
