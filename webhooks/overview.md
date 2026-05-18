# Plivo Webhooks Overview

When events happen on Plivo (message delivered, call ended, OTP verified, recording done), Plivo HTTP-POSTs (or GETs) a URL you configured. This folder catalogs the payloads.

## How URLs get configured

- **App-level** — set `answer_url`, `hangup_url`, `message_url`, etc. on `/Application/`
- **Per-request** — pass `url`, `hangup_url`, `machine_detection_url` in /Message/ or /Call/ POST
- **Resource-level** — Verify, Recording, Stream each have their own callback URLs

## HTTP method & content type

- Default method: POST. Override per-callback with `method=GET`.
- **POST body is `application/x-www-form-urlencoded`, NOT JSON.** Surprising — read form params, not JSON.
- GET passes the same fields as query string.

## Retry behavior

Plivo retries failed (non-2xx) webhooks:
- Message DLR: 3 attempts, 60s / 120s / 240s backoff
- Verify: 3 attempts, same backoff
- Voice callbacks: limited retries

Always return 2xx fast. Process work async if needed.

## Security

ALWAYS verify the signature — see `webhooks/signature.md`. Anyone can POST to your URL otherwise.

## Reference index

- `webhooks/sms.md` — DLR (delivery report) + inbound SMS
- `webhooks/call.md` — hangup, ring, machine detection, fallback
- `webhooks/recording.md` — recording complete + transcription
- `webhooks/stream.md` — audio stream lifecycle + WebSocket message format
- `webhooks/verify.md` — Verify session lifecycle
- `webhooks/signature.md` — V3 signature verification (auth)

## Inbound calls / SMS — note

Inbound calls/SMS aren't strictly async webhooks; they're real-time:
- **Inbound SMS** → Plivo POSTs form-encoded params to your app's `message_url`. Return any 2xx (optionally return Plivo XML `<Message>` to auto-reply).
- **Inbound call** → Plivo fetches your app's `answer_url`, you must return Plivo XML (see xml/core.md). The call IS the connection — your XML controls it.
