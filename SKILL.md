---
name: plivo
description: Use Plivo's REST API for SMS/MMS/WhatsApp messaging, voice calls, OTP verification, phone numbers, conferences, recordings, SIP trunks, subaccounts, and AI voice agents (audio streaming over WebSocket). Trigger when the user mentions Plivo, or asks to send a message, place a call, verify a phone via OTP, build/connect a voice agent, search/buy/release a phone number, manage WhatsApp templates, or check Plivo account state.
---

# Plivo Skill

Plivo is a CPaaS provider for voice, SMS, WhatsApp, and Verify (OTP). All operations are HTTP requests with Basic auth.

## Fast path

For 80% of tasks (send SMS, OTP, make a call, list messages, get balance, search numbers): read `cheatsheet.md` first. One file, ~100 lines, ready-to-copy curl snippets.

For the other 20% (deeper API control, compliance, voice streaming, etc.): lazy-load the relevant `api/<resource>.md` via the Read tool. Do NOT load every api file upfront.

## Before any action

1. Auth ready? — see `auth.md` (needs `PLIVO_AUTH_ID` and `PLIVO_AUTH_TOKEN` env vars)
2. Classify the action by tier — see `safety.md` (Tier 0 read, Tier 1 confirm, Tier 2 explicit-confirm, Tier 3 hand off to user)
3. Is the user asking for something Plivo doesn't expose via API? — check `not-supported.md`

## API resources (lazy-load)

Messaging: `api/messages.md`, `api/whatsapp.md`, `api/10dlc.md`, `api/toll-free-verification.md`, `api/powerpack.md`, `api/media.md`
Voice: `api/calls.md`, `api/conferences.md`, `api/recordings.md`, `api/endpoints.md`, `api/audio-streams.md`
Verify: `api/verify.md`
Numbers: `api/numbers.md`, `api/compliance.md`
Account: `api/subaccount.md`, `api/application.md`, `api/account.md`, `api/sip-auth.md`, `api/pricing.md`
Other: `api/sip-trunking.md`, `api/number-masking.md`

## XML reference (voice flow control)

For voice calls, `answer_url` must return Plivo XML. Lazy-load:
- `xml/core.md` — Response, Speak, Play, Wait, GetInput, GetDigits, DTMF, Hangup, Redirect (90% of needs)
- `xml/advanced.md` — Dial, Number, User, Conference, Record, Stream, Message, PreAnswer, SIPDigits

## Webhook payload reference (incoming events)

When Plivo POSTs events to your URLs (form-encoded, NOT JSON):
- `webhooks/overview.md` — patterns, retries, content-type, where webhooks come from
- `webhooks/sms.md` — DLR + inbound SMS payloads
- `webhooks/call.md` — hangup, ring, AMD, fallback, HangupCause catalog
- `webhooks/recording.md` — recording complete + transcription
- `webhooks/stream.md` — audio stream lifecycle + WebSocket message format
- `webhooks/verify.md` — Verify session events
- `webhooks/signature.md` — V3 HMAC verification (always do this)

## Workflows (multi-step recipes)

`workflows/send-otp.md`, `workflows/send-transactional-sms.md`, `workflows/make-outbound-call.md`, `workflows/send-whatsapp-template.md`, `workflows/subaccount-lifecycle.md`

## Plivo API quirks (don't trip on these)

- **Response array keys vary**: `objects[]` for most, but `calls[]` (live), `conferences[]`, `brands[]`, `campaigns[]`, `compliances[]`, `profiles[]`, `numbers[]` (buy response wraps with `status:"fulfilled"`). Always check the response shape.
- **HTTP status codes vary**: POST creation → 201, POST update (e.g., `/Number/{n}/`) → 202 "changed" (async accept), DELETE → 204. Don't assume 200 == success.
- **`/Masking/Session/`** wraps list in `response{...}` — extra unwrap needed.
- **`/MultiPartyCall/`** uses `meta.count` not `meta.total_count`.
- **`/Verify/Session/` list** returns HTTP 400 `"no attempts found"` if zero sessions — treat 400 as no-data, not error.
- **Money fields are STRINGS** (`"cash_credits":"10"`, `"voice_rate":"0.00880"`). Parse before math.
- **Path naming quirks**: `/TollfreeVerification/` (lowercase `f`!), `/SipAuth/IpAccessControlList/` (not `/AuthIp/`), `/PhoneNumber/Compliance/` (not `/Compliance/Application/`).
- **Application's `log_incoming_message`** is singular (not plural).
- **ACL ID field** is `ip_acl_uuid` (not `uuid` or `ip_access_control_list_uuid`).
- **XML attributes are camelCase, REST JSON body is snake_case** for the SAME params: `<Stream contentType="...">` (XML) vs `{"content_type":"..."}` (REST). Don't mix.

## How to optimize for speed

- Use cheatsheet.md for any task that looks "standard"
- Read api/*.md ONLY when a specific param or behavior isn't in cheatsheet
- Never load all api files; resolve one at a time
