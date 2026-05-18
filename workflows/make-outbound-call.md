# Workflow: Make Outbound Call

## Pre-reqs

- Owned Plivo number OR a Verified Caller ID
- A public HTTPS endpoint returning Plivo XML (the "answer URL")

## Step 1 — Host an answer URL

Your endpoint returns Plivo XML when the call connects:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Response>
  <Speak voice="Polly.Joanna">Hello, this is an automated call.</Speak>
  <Hangup/>
</Response>
```

## Step 2 — Make the call

```bash
curl -X POST "https://api.plivo.com/v1/Account/$PLIVO_AUTH_ID/Call/" \
  -u "$PLIVO_AUTH_ID:$PLIVO_AUTH_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "from":"+1<your_number>",
    "to":"+1<recipient>",
    "answer_url":"https://your.app/answer.xml",
    "answer_method":"GET",
    "ring_timeout":30,
    "hangup_url":"https://your.app/hangup"
  }'
```

Response: `request_uuid` immediately. Real `call_uuid` arrives via your `hangup_url` callback once the call ends.

## Step 3 — Manage during call

- Hang up: `DELETE /Call/{call_uuid}/`
- Transfer (B-leg): `POST /Call/{call_uuid}/` with `legs:"bleg"` and a new XML URL
- Start recording: `POST /Call/{call_uuid}/Record/`
- Start audio stream (AI agent): `POST /Call/{call_uuid}/Stream/` (api/audio-streams.md)

## For AI voice agents

Skip static XML. Use `<Stream>` in answer_url XML to open a WebSocket to your agent:

```xml
<Response>
  <Stream bidirectional="true" keepCallAlive="true">wss://your.agent/ws</Stream>
</Response>
```

Then your agent (Pipecat / LiveKit / Vapi / custom) receives audio and sends synthesized speech back. See api/audio-streams.md.

## Common errors

- "ring_timeout exceeded" — no answer
- "caller ID not verified" — `from` is neither a Plivo number nor a verified caller ID
- "rate limit" — concurrency cap (raise via support)
- Hangup cause `NORMAL_CLEARING` — clean end
- `NO_ANSWER`, `BUSY`, `FAILED`, `CONGESTION` — standard SIP causes
