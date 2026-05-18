# Audio Stream Webhooks + WebSocket Reference

Audio streams have TWO event channels:
1. **HTTP status webhooks** — lifecycle (start/stop/error) POSTed to your `status_callback_url`
2. **WebSocket messages** — the actual audio + control events on the wss:// channel

## HTTP status callbacks

From `<Stream statusCallbackUrl="...">` or POST /Call/{uuid}/Stream/.

Form-encoded params:
- `CallUUID`
- `StreamID`
- `Event` — `start` / `stop` / `media-error`
- `StatusCallbackEvent` — same as Event
- `Timestamp` — ISO 8601
- `ErrorCode`, `ErrorMessage` — only on media-error

## WebSocket messages (Plivo → your agent)

JSON messages on the wss:// connection.

**`start` event:**
```json
{
  "event": "start",
  "streamId": "<stream_id>",
  "start": {
    "accountId": "<auth_id>",
    "streamId": "<stream_id>",
    "callId": "<call_uuid>",
    "tracks": ["inbound"],
    "mediaFormat": {"encoding":"audio/x-mulaw","sampleRate":8000,"channels":1}
  }
}
```

**`media` event** (continuous):
```json
{
  "event": "media",
  "media": {
    "track": "inbound",
    "chunk": "<int>",
    "timestamp": "<ms>",
    "payload": "<base64-encoded-audio>"
  }
}
```

**`dtmf` event:**
```json
{
  "event": "dtmf",
  "dtmf": {"digit":"5"}
}
```

**`stop` event:**
```json
{"event":"stop","stop":{"reason":"..."}}
```

## WebSocket messages (your agent → Plivo, if bidirectional)

**Send audio:**
```json
{"event":"media","media":{"payload":"<base64-encoded-audio>"}}
```

**Clear buffer** (interruption handling):
```json
{"event":"clear","streamId":"<stream_id>"}
```

Useful when caller interrupts — stop in-flight TTS playback.

## Audio format

- Default: `audio/x-mulaw;rate=8000` (G.711 μ-law, 8kHz, mono)
- Optional: `audio/x-l16;rate=16000` (linear PCM, 16kHz)

Match this to your STT/TTS pipeline. Mismatched = silence or distortion.

## Common AI voice agent stack

- **Orchestrator**: Pipecat, LiveKit Agents, Vapi, Plivo Stream SDK
- **STT/LLM/TTS providers**: OpenAI Realtime API, Gemini Live, ElevenLabs Conversational AI, Cartesia, Sarvam (Indian languages)

Plivo has integration guides for each: https://www.plivo.com/docs/voice-agents/audio-streaming/

## Gotchas

- WebSocket must be reachable from Plivo's IPs (firewall the listening port)
- Latency budget: <500ms end-to-end (caller speaks → agent responds) or UX breaks
- For interruption handling, the `clear` event is essential
- Don't buffer audio long on agent side — pipeline forward immediately
- TLS required (wss://), not raw ws://
