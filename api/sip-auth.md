# SIP Authentication

Two resources under `/SipAuth/`:
- **IP Access Control Lists (ACLs)** — IP allowlists for SIP traffic
- **Credentials** — username/password pairs for SIP auth

Docs: https://www.plivo.com/docs/account/api/sip-authentication

> Note: there is NO `/AuthIp/` endpoint. Older docs reference it but the real path is `/SipAuth/IpAccessControlList/...`.

## IP Access Control Lists (ACLs)

| Method | Path | Purpose |
|---|---|---|
| GET | /SipAuth/IpAccessControlList/ | List ACLs |
| POST | /SipAuth/IpAccessControlList/ | Create ACL — Tier 1 |
| GET | /SipAuth/IpAccessControlList/{ip_acl_uuid}/ | Get |
| POST | /SipAuth/IpAccessControlList/{ip_acl_uuid}/ | Update |
| DELETE | /SipAuth/IpAccessControlList/{ip_acl_uuid}/ | Delete — Tier 2 |

Create body:
```json
{"name":"my-acl"}
```
Response includes `ip_acl_uuid` (the ID for downstream calls — NOT `uuid` or `ip_access_control_list_uuid`).

## ACL Entries (IPs in an ACL)

| Method | Path | Purpose |
|---|---|---|
| POST | /SipAuth/IpAccessControlList/{ip_acl_uuid}/Entry/ | Add IP/CIDR — Tier 1 |
| GET | /SipAuth/IpAccessControlList/{ip_acl_uuid}/Entry/ | List entries |
| GET | /SipAuth/IpAccessControlList/{ip_acl_uuid}/Entry/{entry_id}/ | Get |
| DELETE | /SipAuth/IpAccessControlList/{ip_acl_uuid}/Entry/{entry_id}/ | Remove — Tier 2 |

Add body:
```json
{"ip":"1.2.3.4","subnet":"32"}
```
`subnet: "32"` = single IP. Use `24`/`16`/`8` for CIDR ranges. Max 50 entries per ACL.

## SIP Credentials

| Method | Path | Purpose |
|---|---|---|
| GET | /SipAuth/Credential/ | List |
| POST | /SipAuth/Credential/ | Create — Tier 1 |
| GET | /SipAuth/Credential/{credential_uuid}/ | Get |
| POST | /SipAuth/Credential/{credential_uuid}/ | Update |
| DELETE | /SipAuth/Credential/{credential_uuid}/ | Delete — Tier 2 |

Create body:
```json
{"username":"sipuser","password":"<your_password>"}
```
Response includes `credential_uuid` and `realm` (`app.plivo.com`).

## Attach to SIP Endpoints

ACLs and credentials are attached to SIP Endpoints via `/Endpoint/{ep_id}/` update — see api/endpoints.md.

## Gotchas

- Once an ACL is attached to an endpoint, only allowlisted IPs can register — easy lockout. Add dev IPs FIRST.
- ACL `ip_acl_uuid` is the ID — don't confuse with the generic `uuid` pattern other Plivo resources use.
- Credential `realm` is fixed at `app.plivo.com`.
