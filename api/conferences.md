# Conferences & Multiparty Calls

Multi-participant calls. Two flavors:
- **Conferences** — traditional bridge, created via XML `<Conference>` verb, managed via REST
- **Multiparty Calls (MPC)** — modern, REST-driven add/remove

Docs:
- https://www.plivo.com/docs/voice/api/conferences
- https://www.plivo.com/docs/voice/api/multiparty-calls

## Conferences

| Method | Path | Purpose |
|---|---|---|
| GET | /Conference/ | List active |
| GET | /Conference/{name}/ | Get details |
| DELETE | /Conference/{name}/ | Terminate — Tier 1 |
| DELETE | /Conference/{name}/Member/{member_id}/ | Kick member |
| POST | /Conference/{name}/Member/{member_id}/Mute/ | Mute |
| DELETE | /Conference/{name}/Member/{member_id}/Mute/ | Unmute |
| POST | /Conference/{name}/Member/{member_id}/Deaf/ | Make deaf (can't hear) |
| DELETE | /Conference/{name}/Member/{member_id}/Deaf/ | Undeaf |
| POST | /Conference/{name}/Member/{member_id}/Speak/ | Play TTS to member |
| POST | /Conference/{name}/Member/{member_id}/Play/ | Play audio file |
| POST | /Conference/{name}/Record/ | Start recording |
| DELETE | /Conference/{name}/Record/ | Stop recording |

Conferences are CREATED via XML, not REST. Use `<Conference>` verb in your answer_url XML.

## Multiparty Calls (MPC)

| Method | Path | Purpose |
|---|---|---|
| POST | /MultiPartyCall/{name}/Participant/ | Add participant — Tier 1 |
| GET | /MultiPartyCall/ | List MPCs |
| GET | /MultiPartyCall/{name}/ | Get MPC |
| DELETE | /MultiPartyCall/{name}/ | Terminate MPC — Tier 1 |
| GET | /MultiPartyCall/{name}/Participant/ | List participants |
| DELETE | /MultiPartyCall/{name}/Participant/{id}/ | Remove |
| POST | /MultiPartyCall/{name}/Participant/{id}/Mute/ |
| DELETE | /MultiPartyCall/{name}/Participant/{id}/Mute/ |
| POST | /MultiPartyCall/{name}/Record/ | Start recording |
| DELETE | /MultiPartyCall/{name}/Record/ | Stop |

Add participant body:
```json
{"from":"+1XXX","to":"+1YYY","role":"agent","call_status_callback_url":"..."}
```

## Which to use

- MPC for new builds (REST-only, no XML, easier to reason about programmatically)
- Conferences if you have existing XML-based call flows
