# Powerpack

Pool of source numbers with smart routing (geo-matched, sticky sender, round-robin).

Docs: https://www.plivo.com/docs/messaging/api/powerpack

## Powerpack lifecycle

| Method | Path | Purpose |
|---|---|---|
| POST | /Powerpack/ | Create — Tier 2 |
| GET | /Powerpack/ | List |
| GET | /Powerpack/{uuid}/ | Get |
| POST | /Powerpack/{uuid}/ | Update |
| DELETE | /Powerpack/{uuid}/ | Delete |

Create params: `name`, `application_id`, `sticky_sender` (bool), `local_connect` (bool), `number_priority`.

## Number pool

| Method | Path | Purpose |
|---|---|---|
| GET | /Powerpack/{uuid}/NumberPool/ | List numbers in pool |
| POST | /Powerpack/{uuid}/NumberPool/Number/{number}/ | Add number |
| DELETE | /Powerpack/{uuid}/NumberPool/Number/{number}/ | Remove |
| GET | /Powerpack/{uuid}/NumberPool/Number/ | List with filters |
| POST | /Powerpack/{uuid}/NumberPool/FindNumber/ | Auto-add numbers matching criteria |

## Use in send

```json
POST /Message/
{"powerpack_uuid":"<uuid>","dst":"+1YYY","text":"..."}
```
Plivo picks the best source from the pool.

## Why use it

- Higher throughput than single source
- Local-area-code matching for better deliverability
- Sticky pairing (same recipient → same sender)
- A2P 10DLC: pool numbers must all be in the same campaign
