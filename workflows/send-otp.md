# Workflow: Send & Validate OTP

End-to-end OTP flow via Plivo Verify.

## Pre-reqs

- A Verify App created in Plivo Console → Verify → Apps → get `app_uuid`
- Recipient phone in E.164 (e.g., `+14155551234`)

## Step 1 — Start session

```bash
curl -X POST "https://api.plivo.com/v1/Account/$PLIVO_AUTH_ID/Verify/Session/" \
  -u "$PLIVO_AUTH_ID:$PLIVO_AUTH_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "recipient":"+1234567890",
    "app_uuid":"<VERIFY_APP_UUID>",
    "channel":"sms"
  }'
```

Save `session_uuid` from response.

## Step 2 — User receives OTP, enters it

Your app collects the 6-digit code from the user (UI, IVR, etc.).

## Step 3 — Validate

```bash
curl -X POST "https://api.plivo.com/v1/Account/$PLIVO_AUTH_ID/Verify/Session/<session_uuid>/" \
  -u "$PLIVO_AUTH_ID:$PLIVO_AUTH_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"otp":"123456"}'
```

`status: verified` → success.

## Optional: voice channel

Pass `"channel":"voice"` to read code aloud instead of SMS.

## Common errors

- 422 "session not found" — wrong session_uuid or session expired (default TTL 600s)
- 422 "max attempts" — 10 wrong tries, start a new session
- 422 "OTP expired" — past TTL, start a new session
- 422 "Recipient blocked" — brute-force lockout

## Notes

- Plivo handles random code generation (don't pass your own code)
- One active session per recipient at a time
- Status callback `url` arrives async with delivery + verification events
