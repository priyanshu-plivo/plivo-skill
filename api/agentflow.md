# Agents (AgentFlow)

No-code conversation agents built as a graph: a Start trigger plus nodes (`send_message`, `http_request`, `ai_agent_v2`, `branch`, ...) wired by connections. The same agents the console's AI agent builder edits. Beta: shapes may change.

CLI equivalent: `plivo agents` (https://www.plivo.com/docs/cli/agents)

## Endpoints

Paths are case-sensitive (`/agentflow/` is a 404). Subaccount credentials get 403, so call as the main account.

| Method | Path | Purpose |
|---|---|---|
| GET | /AgentFlow/ | List. `limit` (max 20), `offset`, `name` (contains), `state` (`DRAFT`, `ACTIVE`, `PAUSED`) |
| POST | /AgentFlow/ | Create, lands in `DRAFT` (Tier 1, HTTP 201) |
| GET | /AgentFlow/{agent_uuid}/ | Get, with `nodes` and `connections` |
| POST | /AgentFlow/{agent_uuid}/ | Update, replaces the whole graph (Tier 1 on a `DRAFT`, Tier 2 on a live agent; HTTP 202) |
| DELETE | /AgentFlow/{agent_uuid}/ | Delete (Tier 2, HTTP 204, permanent) |
| POST | /AgentFlow/{agent_uuid}/Publish | `DRAFT` to `ACTIVE`, starts taking traffic (Tier 2, HTTP 202, no body) |
| POST | /AgentFlow/{agent_uuid}/Pause | `ACTIVE` to `PAUSED`, stops taking traffic (Tier 2, HTTP 202, no body) |
| POST | /AgentFlow/{agent_uuid}/Resume | `PAUSED` to `ACTIVE` (Tier 2, HTTP 202, no body) |
| GET | /AgentFlow/{agent_uuid}/Run/ | Run history. `limit` (max 20), `offset` |
| GET | /AgentFlow/{agent_uuid}/Run/{run_uuid}/ | One run, plus `logs` and `goal_metrics` |
| GET | /AgentFlowNode/ | Node type catalogue: `node_type`, `category`, `usecase`, `output_states`. `?types=a,b` returns several |
| GET | /AgentFlowNode/{node_type}/ | One type: `json_schema` for `config.model`, `output_states`, `examples` |

## Rules (break one and the agent still saves, then misbehaves)

1. **Set every node's `id` yourself, to a fresh UUID v4.** One per node (`uuidgen | tr A-Z a-z`), the format console-built agents use. Never omit `id` (the server then assigns `<type>_<n>`, like `send_message_1`) and never copy ids from `examples` (`start-1`, `reply-1`) or from another agent: those repeat across agents. Once saved, never change an id.
2. **Never invent `config.model`.** Follow the type's `json_schema` from `GET /AgentFlowNode/{type}/`. Its `examples` are complete flows for a few types (give their nodes new ids, rule 1).
3. **Connection = `{"source": "<node_id>.<handle>", "target": "<node_id>.Input"}`.** Source handles are the source type's `output_states[].id`, and a `branch` also gets one per `conditions[].alias`. Handles are not validated: a typo saves (201), publishes, and that path never fires.
4. **Connections use node `id`; template variables use node `name`:** `{{Start.message.from}}`, never `{{<id>.message.from}}`. Renaming a node means updating its variables in the same request.
5. **Every node must appear in a connection** (422 names any that don't). Wire everything from Start: a node with no incoming connection saves but never runs.
6. **Drafts save with invalid config.** Errors surface only at Publish (422 naming node and field), so check each node against its schema first.

## Create

```bash
S=$(uuidgen | tr A-Z a-z); R=$(uuidgen | tr A-Z a-z)   # one fresh id per node
curl -s -u "$PLIVO_AUTH_ID:$PLIVO_AUTH_TOKEN" -H "Content-Type: application/json" \
  --data-binary @- "https://api.plivo.com/v1/Account/$PLIVO_AUTH_ID/AgentFlow/" <<EOF
{"name": "SMS auto-reply",
 "nodes": [
  {"id": "$S", "name": "Start", "type": "start",
   "config": {"model": {"triggers": ["message"]}}},
  {"id": "$R", "name": "Send Message", "type": "send_message",
   "config": {"model": {"from": "$PLIVO_NUMBER", "to": ["{{Start.message.from}}"],
                        "text": "Thanks, we will reply shortly."}}}],
 "connections": [{"source": "$S.message", "target": "$R.Input"}]}
EOF
```

Response (HTTP 201): `{"api_id": "...", "message": "agent created", "agent_uuid": "...", "name": "SMS auto-reply", "resource_uri": "..."}`

- `PLIVO_NUMBER` is a number on the account (E.164). Keep the heredoc: inline `-d '...'` JSON mixed with shell variables breaks easily.
- `name` is required. If it is taken, the stored name gets a number appended, so read `name` from the response.
- `flow_type` (`incoming_sms`, `incoming_call`, `api_request`, ...) comes from the Start trigger unless you send it, and never changes after create.
- Keep the Start node's `triggers` in step with the handle its connection uses (`["message"]` with `"<start_id>.message"`).
- Then publish (Tier 2, it goes live): `POST /AgentFlow/{agent_uuid}/Publish` with no body.

## Update

POST the whole graph: read, edit, send back.

```bash
B="https://api.plivo.com/v1/Account/$PLIVO_AUTH_ID/AgentFlow/$AGENT_UUID/"
curl -s -u "$PLIVO_AUTH_ID:$PLIVO_AUTH_TOKEN" "$B" > agent.json
# edit agent.json: keep every id as it came back, give each new node a fresh UUID
python3 -c "import json; d=json.load(open('agent.json')); json.dump({'nodes': d['nodes'], 'connections': d['connections']}, open('body.json', 'w'))"
curl -s -u "$PLIVO_AUTH_ID:$PLIVO_AUTH_TOKEN" -H "Content-Type: application/json" --data-binary @body.json "$B"
```

Response (HTTP 202): `{"api_id": "...", "message": "changed"}`

- Send `nodes` and `connections` together. Without `connections` there are no edges, so every node fails as unconnected (422).
- Leaving out a stored node is a 422 that names it. To delete nodes on purpose, add `"allow_node_deletion": true` and drop their connections.
- A node's `type` cannot change (422). Add a new node instead.
- `{"name": "..."}` on its own renames. `description` is only accepted together with `nodes` (400 otherwise).
- On a live agent the change hits new runs at once: no staging, last write wins. A 409 means a Vibe Agent run is editing it; retry after.
- Reads mask secret-shaped values as `***REDACTED***`. Sent back unchanged, the stored secret is kept. A masked value on create is a 400, so a GET cannot clone an agent that holds secrets.
- Some types read back with a nested field group (an `http_request` node's fields sit under `model.http_request`). Posting that back as read is fine; write new config flat, as `json_schema` describes.

## Lifecycle

- Only a `DRAFT` can be published. Publishing an `ACTIVE` or `PAUSED` agent is a 422, and so is a draft with saved config errors.
- Pause and Resume refuse a `DRAFT` (422): use Publish.

## Runs

`status` is `QUEUED`, `IN_PROGRESS`, `COMPLETED`, `FAILED` or `EXPIRED`. Fields: `run_uuid`, `agent_uuid`, `status`, `started_at`, `ended_at`, `duration` (seconds), `is_playground`. The run detail adds `logs` (time-ordered events whose fields vary by node, not a stable contract) and `goal_metrics` (`null`, or `{"goal_evaluations": [{"goal_name", "goal_achieved", "reason"}]}`).

## Common handles

| Node type | Source handles |
|---|---|
| `start` | `message`, `call`, `chat`, `whatsapp`, `email`, `http`, `outbound_call`, `outbound_message`, `outbound_whatsapp`, `outbound_email`, `destination`, `call_hangup`, `incoming_slack` |
| `send_message` | `sent`, `failed` |
| `http_request` | `success`, `failed` |
| `ai_agent_v2` | `sent`, `error` |
| `call_forward` | `completed`, `no_answer`, `busy_rejected`, `failed` |
| `branch` | `no_match`, plus each `conditions[].alias` |

`hangup`, `end_conversation`, `queue` and `ai_action` have none: they end a path. For any other type, read `output_states` from the catalogue.

## Errors

Body: `{"api_id": "...", "error": "<message>"}`

| Status | Meaning |
|---|---|
| 400 | Bad parameter, body not a JSON object or over 2 MB, endpoint not `<node_id>.<handle>`, `agent_uuid` not a UUID, unknown type in `?types=` |
| 401 | Bad credentials, or the path `auth_id` is not the account you authenticated as |
| 403 | Subaccount credentials, or the account is not enabled for Agents |
| 404 | No such agent, run or node type |
| 409 | A Vibe Agent run is editing the agent; retry when it ends |
| 422 | Understood but not applied: unconnected node, dropped node without `allow_node_deletion`, type change, failed validation, wrong state for Publish/Pause/Resume |
| 429 | Rate limited; wait for `Retry-After` |

## Gotchas

- `limit` above 20 is clamped to 20 (agents, runs), not rejected; read `meta.limit`. The node catalogue pages at 100, which covers it.
- Caps: 500 nodes and 2,000 connections per agent.
- Keys are `agent_uuid` and `run_uuid`. Timestamps look like `2026-07-28 04:24:02.000000+00:00`, not ISO `T`/`Z`.
- `version` is always `1`; compare `updated_at` to detect edits.
- List rows and the create response carry no nodes; `GET /AgentFlow/{agent_uuid}/` does.
