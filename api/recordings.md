# Recordings

Manage call recordings. Recording starts via XML `<Record>` verb OR REST start/stop on active call.

Docs: https://www.plivo.com/docs/voice/api/recordings

## Endpoints

| Method | Path | Purpose |
|---|---|---|
| GET | /Recording/ | List |
| GET | /Recording/{recording_id}/ | Get |
| DELETE | /Recording/{recording_id}/ | Delete — Tier 1 |
| POST | /Call/{call_uuid}/Record/ | Start recording active call — Tier 1 |
| DELETE | /Call/{call_uuid}/Record/ | Stop |

## Start params (POST)

- `time_limit` — max seconds (default 3600)
- `transcription_type` — `auto` or `hybrid`
- `transcription_url` — webhook for transcript
- `callback_url` — webhook on stop
- `callback_method`
- `file_format` — `mp3` (default) or `wav`

## List filters

`call_uuid`, `from_number`, `to_number`, `add_time__gte`, `add_time__lt`, `limit`, `offset`.

## Storage

Recordings stored on Plivo CDN. URL in `recording_url` field. Retention configurable in Console (default ~30 days; longer plans available).

## Gotchas

- Recording consent: legal requirements vary by jurisdiction (US two-party consent states, GDPR, etc.) — your responsibility
- Stopping recording mid-call: don't expect partial; full recording up to stop point is saved
- Transcription is async; webhook arrives after recording completes
