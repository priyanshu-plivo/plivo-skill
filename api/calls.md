# Voice Calls

Outbound calls, transfer, hangup, CDRs, active call control.

Docs: https://www.plivo.com/docs/voice/api/calls

## Endpoints

| Method | Path | Purpose |
|---|---|---|
| POST | /Call/ | Make outbound call |
| GET | /Call/ | List CDRs (completed) |
| GET | /Call/{call_uuid}/ | Get CDR |
| GET | /Call/?status=live | List active |
| GET | /Call/{call_uuid}/?status=live | Get active call |
| POST | /Call/{call_uuid}/ | Update/transfer active |
| DELETE | /Call/{call_uuid}/ | Hang up |

## Make a call

Required: `from` (your Plivo number or verified caller ID), `to`, `answer_url`.

```json
{
  "from": "+1XXX",
  "to": "+1YYY",
  "answer_url": "https://your.app/answer.xml",
  "answer_method": "GET",
  "hangup_url": "https://your.app/hangup",
  "ring_timeout": 30,
  "time_limit": 14400,
  "machine_detection": "true",
  "machine_detection_url": "https://your.app/amd",
  "caller_name": "Plivo Inc"
}
```

`answer_url` must return Plivo XML to control the call. See https://www.plivo.com/docs/voice/xml/overview.

## Additional optional params

- `ring_url`, `ring_method` — webhook when ringing starts
- `fallback_url`, `fallback_method` — used if `answer_url` errors
- `send_digits` — DTMF digits to send after answer (IVR navigation)
- `send_on_preanswer` — bool, send digits pre-answer
- `sip_headers` — extra SIP headers (`X-Custom1=v1,X-Custom2=v2`)
- `sip_auth_username`, `sip_auth_password` — SIP destination auth
- Multiple destinations: use `<` between `to` values (max 1000)

## Transfer / update active call

```json
POST /Call/{call_uuid}/
{"legs":"aleg","aleg_url":"https://your.app/transfer.xml","aleg_method":"GET"}
```

## Verified Caller IDs

If `from` is NOT your Plivo number, it must be a verified caller ID.

| Method | Path | Purpose |
|---|---|---|
| GET | /VerifiedCallerId/ | List |
| POST | /VerifiedCallerId/ | Initiate verification (sends OTP via SMS/voice) — Tier 1 |
| DELETE | /VerifiedCallerId/{cid_uuid}/ | Remove |

Verification itself is manual (user picks up the OTP call/SMS) — see not-supported.md.

## Gotchas

- `time_limit` max 14400s (4 hours)
- `machine_detection`: adds 4–6s latency; use only if you handle voicemail differently
- India: STIR/SHAKEN compliance required; see compliance.md
- Concurrent call limit defaults are region-dependent; raise via support
- POST response gives `request_uuid` immediately; `call_uuid` arrives in hangup callback or via GET /Call/?status=live
- **HTTP 403 `"Calls to this destination region are barred."`** — Plivo's Geo-Permissions block international calls by default. Enable target countries in Console → Voice → Geo-Permissions before calling. There is NO REST API for managing geo-permissions — Console-only (also see not-supported.md).
- Default Plivo `from` numbers are restricted by carrier — must be a Plivo-owned number or Verified Caller ID
