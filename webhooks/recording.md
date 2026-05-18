# Recording Webhooks

POSTed when a recording completes (started by `<Record>` XML verb or POST /Call/{uuid}/Record/).

## Recording callback (action URL)

URL from `<Record action="...">` or REST `callback_url`.

Form-encoded params:
- `CallUUID`
- `RecordingID` — Plivo's recording UUID
- `RecordUrl` — URL to download the audio (MP3 or WAV)
- `RecordingDuration` — seconds (integer)
- `RecordingDurationMs` — milliseconds
- `RecordingStartMs` — offset in call when recording began
- `RecordingEndMs` — offset when ended
- `Digits` — DTMF that ended recording (if `finishOnKey` triggered)
- `Event` — `Record`

## Transcription callback (separate URL)

Only fires if `transcriptionType: auto` or `hybrid` was set on `<Record>`.

URL from `<Record transcriptionUrl="...">`.

Form-encoded params:
- `CallUUID`
- `RecordingID`
- `TranscriptionStatus` — `success` / `failed`
- `TranscriptionRate` — per-minute rate (STRING)
- `TranscriptionCharge` — total cost (STRING)
- `TranscriptionId`
- `TranscriptionUrl` — fetch THIS URL for the actual transcript text

Transcript text isn't in the webhook — GET the `TranscriptionUrl` separately.

## Downloading the recording

`RecordUrl` is fetched with the same HTTP Basic auth as the REST API (see `auth.md`).

```bash
curl --user "$PLIVO_AUTH_ID:$PLIVO_AUTH_TOKEN" -o recording.mp3 "$RecordUrl"
```

Retention: configurable in Console (default ~30 days; longer plans available).

## Gotchas

- Transcription is async — may arrive minutes after the recording callback
- For sensitive content: copy to your storage, then DELETE /Recording/{id}/ via REST
- `RecordingDuration` is rounded; use `RecordingDurationMs` for billing accuracy
- Recording may fail silently — if you don't get the callback within X minutes, check via GET /Recording/?call_uuid=...
