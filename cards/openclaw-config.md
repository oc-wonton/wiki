# OpenClaw Config Notes

## Key settings for agents

### maxSpawnDepth
- `1` (default): Two layers only (coordinator + subagent)
- `2`: Three layers (L0 → L1 → L2), required for [[m2-pattern]]
- Location: `agents.defaults.subagents.maxSpawnDepth`

### maxConcurrent
- How many subagents can run in parallel
- Current: `8` (subagents), `4` (main agent concurrent)

### Model
- Current primary: `github-copilot/claude-opus-4.6`

## Secrets
- `openclaw secrets audit` — check for plaintext secrets
- `openclaw secrets configure` — interactive setup
- Token/secrets should use SecretRef, not plaintext

## Skills
- `openclaw skills list` — show all available skills
- Skills come from: openclaw-bundled, openclaw-extra, local overrides
- Local skills: `~/.openclaw/skills/`

## Useful commands
- `openclaw status` — channel health and sessions
- `openclaw config get <path>` — read config value
- `openclaw config set <path> <value>` — write config value
- `openclaw doctor` — health checks + quick fixes

---
Tags: #openclaw #config #setup
Links: [[m2-pattern]] [[github-api]]
