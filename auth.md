# Auth

Plivo uses HTTP Basic Auth. Username = Auth ID, password = Auth Token.

## Env vars (required)

```
PLIVO_AUTH_ID=MAxxxxxxxxxxxxxxxxxx
PLIVO_AUTH_TOKEN=xxxxxxxxxxxxxxxxxxxxxxxx
```

Get from https://cx.plivo.com/home.

## Base URL

```
https://api.plivo.com/v1/Account/{auth_id}/
```

## Example

```bash
curl -u "$PLIVO_AUTH_ID:$PLIVO_AUTH_TOKEN" \
  https://api.plivo.com/v1/Account/$PLIVO_AUTH_ID/
```

## Subaccount auth

To act as a subaccount, use that subaccount's own `auth_id` + `auth_token`. OR call from the parent with the subaccount's `auth_id` in the URL (works because parent has implicit access).

## Errors

- `401 Unauthorized` — bad creds, or Auth Token was rotated
- `403 Forbidden` — IP not in allowlist (if `sip-auth` IP allowlist is enabled)
- `404 "Account not found"` — `auth_id` in URL doesn't match the creds used

## Security

- Never commit auth creds. Use env vars or a secret manager.
- Rotate Auth Token if exposed — done manually in the Console (no API for rotation, see not-supported.md).
- Restrict by IP via the AuthIp endpoints (api/sip-auth.md).
