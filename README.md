# plivo-skill

A Claude skill for using Plivo's REST API from any agent — SMS, voice, WhatsApp, Verify, numbers, accounts.

## What it is

A directory of markdown files that teaches Claude to call Plivo APIs directly. Drop into `~/.claude/skills/plivo/` (or anywhere Claude reads skills) and Claude will use it whenever the user asks about Plivo.

No code, no MCP server, no separate process. Just markdown.

## Install

```bash
# global (all projects)
cp -r plivo-skill ~/.claude/skills/plivo

# project-local
cp -r plivo-skill .claude/skills/plivo
```

Set env vars in your shell:
```bash
export PLIVO_AUTH_ID=MAxxxxxxxxxxxxxxxxxx
export PLIVO_AUTH_TOKEN=xxxxxxxxxxxxxxxxxxxxxxxx
```

Get these from https://console.plivo.com/dashboard/.

## Layout

```
plivo-skill/
├── SKILL.md             entry point (read first by Claude)
├── cheatsheet.md        top-80% ops, one page (fast path)
├── auth.md              auth + env vars
├── safety.md            agent-vs-human tiers
├── not-supported.md     what Plivo doesn't expose via API
├── api/                 per-resource detailed references
└── workflows/           multi-step recipes
```

## Optimization

This skill is designed for low context cost:

- **SKILL.md is ~30 lines** — small load when skill triggers
- **cheatsheet.md (~100 lines)** covers ~80% of tasks in one Read
- **api/*.md are lazy-loaded** — only read the one(s) you need
- **Tables over prose, curl over English** — max info density
- **Frontmatter triggers** auto-invoke only on Plivo-related intents

Typical cost per task: 150–300 lines of context, vs ~2000 lines if loaded monolithically.

## Coverage

All Plivo REST API resources documented at https://plivo.com/docs/.

## What's NOT covered

Manual / console-only operations (account recharge, number porting, KYC, WhatsApp template creation, WABA onboarding, team management). See `not-supported.md` for the full list.

## CLI follow-up

This skill is markdown-only — it cannot enforce safety (just instructs Claude to ask). For real enforcement of spend caps, confirmation gates, and dry-run mode, build a thin CLI that wraps the same operations and have this skill call the CLI for any mutation. The skill remains the agent-facing manual.
