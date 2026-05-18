# Audio Streaming

Real-time bidirectional audio via WebSocket. The primary path for AI voice agents.

Docs: https://www.plivo.com/docs/voice/api/audio-streams

## Endpoints

| Method | Path | Purpose |
|---|---|---|
| POST | /Call/{call_uuid}/Stream/ | Start stream on active call — Tier 1 |
| GET | /Call/{call_uuid}/Stream/ | List active streams |
| GET | /Call/{call_uuid}/Stream/{stream_id}/ | Get |
| DELETE | /Call/{call_uuid}/Stream/{stream_id}/ | Stop |

## Start params

Required:
- `service_url` — your WebSocket endpoint (`wss://...`)

Optional:
- `audio_track` — `inbound`, `outbound`, or `both`
- `bidirectional` — `true` to allow sending audio back
- `content_type` — `audio/x-mulaw;rate=8000` (default) or `audio/x-l16;rate=16000`
- `stream_timeout` — max stream duration in seconds
- `status_callback_url`, `status_callback_method` — stream lifecycle webhook
- `extra_headers` — custom headers sent on the WebSocket upgrade
- `noise_cancellation` — enable Plivo-side noise reduction
- `noise_cancellation_level` — intensity (integer)

## Use case: AI voice agent

1. Inbound call hits your `answer_url`
2. Return XML that includes `<Stream url="wss://your.agent/" bidirectional="true"/>`
3. Plivo opens WebSocket to your agent
4. Agent receives caller audio, runs STT → LLM → TTS, sends audio back
5. Plivo plays it to the caller

Common orchestrator stacks (Plivo has guides for each):
- **Pipecat** + OpenAI Realtime / Gemini Live / ElevenLabs / Cartesia / Sarvam
- **LiveKit Agents**
- **Vapi**
- **ElevenLabs Conversational AI**

See https://www.plivo.com/docs/voice-agents/audio-streaming/.

## Alternative

Configurable via XML `<Stream>` verb too — no REST call needed if you set up the stream in answer_url XML.

## Gotchas

- WebSocket must be reachable from Plivo's IP range (firewall the listening port appropriately)
- Audio format mismatch = silence or distortion; match `content_type` to your STT/TTS pipeline
- Latency budget: aim for <500ms agent response or it feels broken
