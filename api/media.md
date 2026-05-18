# Media (MMS Upload)

Upload media files for MMS. Referenced by `media_ids` in /Message/ POST.

Docs: https://www.plivo.com/docs/messaging/api/media

## Endpoints

| Method | Path | Purpose |
|---|---|---|
| POST | /Media/ | Upload (multipart/form-data) — Tier 1 |
| GET | /Media/ | List uploaded |
| GET | /Media/{media_id}/ | Get metadata |

No DELETE endpoint documented — unused media auto-expires.

## Upload

`Content-Type: multipart/form-data` (not JSON — different from rest of Plivo API).

```bash
curl -X POST "https://api.plivo.com/v1/Account/$PLIVO_AUTH_ID/Media/" \
  -u "$PLIVO_AUTH_ID:$PLIVO_AUTH_TOKEN" \
  -F "file=@/path/to/image.jpg" \
  -F "file=@/path/to/image2.png"
```

Up to 10 files per request. Each ≤ 2MB.

Response includes per-file: `media_id`, `content_type`, `file_name`, `size`, `media_url`, `upload_time`.

## Use in MMS send

```json
POST /Message/
{
  "src":"+1XXX","dst":"+1YYY","type":"mms",
  "media_ids":["<media_id_1>","<media_id_2>"]
}
```

Alternative without pre-upload: `"media_urls":["https://your.cdn/image.jpg"]` (public HTTPS).

## Retention

- Unused (not sent in MMS): auto-deleted after **6 hours**
- Used in MMS: retained **1 year**

## Gotchas

- Multipart, not JSON — different from rest of Plivo API
- Free uploads, but they expire fast if not used — don't upload then wait days to send
- For frequently-changing media, `media_urls` may be simpler than pre-upload
- 2MB hard limit per file — for larger, host elsewhere and use `media_urls`
