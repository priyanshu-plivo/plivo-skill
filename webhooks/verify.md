# Verify Session Webhooks

Lifecycle events POSTed to the `url` from POST /Verify/Session/.

## Form-encoded params

- `Date` — ISO 8601
- `AppUUID` — Verify App
- `SessionUUID`
- `Recipient` — destination phone
- `Channel` — `sms` / `voice`
- `Status` — `verified` / `expired` / `failed` / `in-progress`
- `RequestUUID` — initial creation
- `MessageUUID` (sms channel) or `CallUUID` (voice channel)
- `Charge` — cost (STRING)
- `Currency` — usually `USD`
- `Reason` — failure description if applicable

## When events fire

- Session created → `in-progress`
- OTP delivered to recipient
- Validation success → `verified`
- Validation failure (max attempts) → `failed`
- TTL exceeded → `expired`

## Retry behavior

Plivo retries 3× with backoff: 60s, 120s, 240s. Return 2xx fast.

## Use case

- Sync OTP state to your DB so retry attempts in your UI know the current status
- Alert on excessive `failed` from same recipient (brute-force indicator)
- Track cost per session via `Charge` for chargeback or rate-limit

## Gotchas

- Webhooks aren't strictly ordered — `verified` could arrive before `in-progress` if delivery is fast
- `Charge` is STRING — parse before sum
- One Verify session can produce 2-3 webhook events (created → delivered → verified)
