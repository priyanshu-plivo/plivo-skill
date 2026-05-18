# Webhook Signature Verification

Plivo signs every webhook so you can verify it actually came from Plivo. Without this, an attacker who knows your webhook URL could POST forged events (fake "delivered" status, fake "verified" OTP, etc.).

Docs: https://www.plivo.com/docs/voice/concepts/signature-validation

## Headers Plivo sends

- `X-Plivo-Signature-V3` — HMAC-SHA256 signature (current)
- `X-Plivo-Signature-V3-Nonce` — random per-request nonce
- `X-Plivo-Signature-V2` — legacy, on older accounts

## Verification algorithm (V3)

1. Build the signed payload string: `<full_webhook_url>` + `<nonce>` + body
   - For POST: body = raw form-encoded body
   - For GET: body = empty string
   - `<full_webhook_url>` must EXACTLY match what Plivo called (path, query string, port if non-standard)
2. Compute HMAC-SHA256 using your **Auth Token** as the key
3. Base64-encode the digest
4. Constant-time compare to `X-Plivo-Signature-V3` header

## Example (Python)

```python
import hmac, hashlib, base64

def verify_plivo_v3(url, nonce, body, signature_header, auth_token):
    payload = url + nonce + body
    expected = base64.b64encode(
        hmac.new(auth_token.encode(), payload.encode(), hashlib.sha256).digest()
    ).decode()
    return hmac.compare_digest(expected, signature_header)
```

## Example (Node)

```js
const crypto = require('crypto');

function verifyPlivoV3(url, nonce, body, sig, authToken) {
  const expected = crypto
    .createHmac('sha256', authToken)
    .update(url + nonce + body)
    .digest('base64');
  return crypto.timingSafeEqual(Buffer.from(expected), Buffer.from(sig));
}
```

## Gotchas

- `url` must include `https://` AND any query string AND non-standard port. CDN/WAF that rewrites path = signature fails.
- `body` must be the RAW form-encoded body — DO NOT parse and re-serialize (key order may differ)
- Use constant-time comparison (`hmac.compare_digest` / `crypto.timingSafeEqual`) — string-equality leaks timing
- For GET requests, body is empty string `""`
- Auth Token rotation invalidates in-flight signatures (rare race condition)
- If your reverse proxy strips `X-Plivo-Signature-V3` headers — fix the proxy config; never disable verification as a workaround
