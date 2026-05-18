# Number Masking

Connect two parties via temporary masked numbers (privacy proxy). Like Uber driver/rider calls.

Docs: https://www.plivo.com/docs/number-masking/concepts/overview

## Sessions

| Method | Path | Purpose |
|---|---|---|
| POST | /Masking/Session/ | Create session — Tier 1 |
| GET | /Masking/Session/ | List |
| GET | /Masking/Session/{session_uuid}/ | Get |
| POST | /Masking/Session/{session_uuid}/ | Update |
| DELETE | /Masking/Session/{session_uuid}/ | Terminate |

## Create

```json
{
  "first_party_phone": "+1XXX",
  "second_party_phone": "+1YYY",
  "virtual_number": "+1ZZZ",
  "session_expiry": 3600,
  "call_record": true,
  "record_file_format": "mp3"
}
```

`virtual_number` is one of your rented Plivo numbers. Calls/SMS to it route based on session participants.

## Use case

- Marketplaces (driver/rider, host/guest, buyer/seller)
- Temporary 1:1 communication where you don't want either party to know the other's real number
- Session auto-expires; communication blocked after

## Gotchas

- Each session reserves the virtual number; consider a pool of virtuals for scale
- Both directions (calls and SMS) routed through the session
- Recording optional but adds cost
