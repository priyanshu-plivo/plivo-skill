# Messages API (SMS / MMS)

Send and retrieve SMS and MMS. WhatsApp uses the same endpoint with `type: whatsapp` — see `whatsapp.md` for templates.

Docs: https://www.plivo.com/docs/messaging/api/messages

## Endpoints

| Method | Path | Purpose |
|---|---|---|
| POST | /Message/ | Send SMS or MMS |
| GET | /Message/ | List messages |
| GET | /Message/{message_uuid}/ | Get one message |
| GET | /Message/{message_uuid}/Media/ | List MMS media |

## Send (POST) — params

Required:
- `src` — sender (E.164 number or alphanumeric sender ID)
- `dst` — destination(s); multiple separated by `<`
- `text` — message body

Optional:
- `type` — `sms` (default), `mms`, or `whatsapp`
- `media_urls` — array of public URLs (MMS)
- `media_ids` — array of pre-uploaded Plivo media UUIDs (MMS)
- `url` — webhook for delivery report
- `method` — webhook HTTP method (default POST)
- `log` — `false` to redact body from logs (PII)
- `trackable` — `true` to enable click tracking
- `powerpack_uuid` — use a Powerpack instead of `src`
- `dlt_entity_id`, `dlt_template_id` — required for India
- `message_expiry` — TTL in seconds before discard

## List filters

`message_state`, `message_direction`, `from_number`, `to_number`, `message_time__gte`, `message_time__lt`, `limit`, `offset`.

## Cost

Per-segment pricing. Segment length:
- GSM-7 encoding: 160 chars per segment
- UCS-2 (any non-GSM char, e.g. emoji): 70 chars per segment
- Concatenated: ~7 fewer chars per segment due to UDH header

## Gotchas

- US: source number must belong to a registered 10DLC campaign or send fails with error 117
- India: `dlt_entity_id` and `dlt_template_id` mandatory; alphanumeric sender ID required
- Long messages: cost is per-segment, not per-message
- POST response only confirms acceptance; final delivery comes via DLR webhook
- Default per-source concurrency: 1 msg/sec (raise via support)
- `log: false` is critical for OTP / sensitive content (PCI/HIPAA)
- **HTTP 403 `"Geo-permission to the destination country is not enabled"`** — Plivo blocks international SMS by default per country. Enable via Plivo Console → Messaging → Geo-Permissions (no API). Same restriction model as voice (see api/calls.md).
