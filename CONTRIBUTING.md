# Contributing

Issues and PRs welcome.

## Workflow

1. Open an issue describing the change
2. Fork, branch (`feat/...` / `fix/...` / `chore/...`)
3. Push and open PR against `main`

## Validating skill metadata

Before submitting, run:
```bash
gh skill publish
```
This validates SKILL.md frontmatter (requires recent `gh` CLI).

## API claims must be verified

All endpoint paths, request params, and response shapes should be verified against:
- The live Plivo API (HTTP request + response), OR
- Current docs at https://www.plivo.com/docs

If you find that this skill's claim doesn't match reality, please open an issue with the request and the actual response.

## Style

- Tables over prose for endpoint references
- Curl examples ready-to-copy (no placeholders that won't actually run)
- Lazy-load pattern: keep SKILL.md small (~50 lines); put depth in `api/*.md`
- One file per resource group
- Document Plivo quirks (response shape variance, naming inconsistencies) in SKILL.md's quirks section

## Reporting bugs

If a documented endpoint returns 404 or behaves differently than documented, please open an issue with:
- The exact curl command
- The actual response (status + body)
- What the skill said it should do

## Out of scope

- WhatsApp Business Account onboarding (Console-only)
- Account recharge / billing (no API)
- See [not-supported.md](./not-supported.md) for the full list

These limitations exist because Plivo doesn't expose them via API — not because we haven't documented them.
