# firmafind Claude Code Plugin

Search, verify and retrieve structured Austrian company data
(Firmenbuch/company register, financials, filings, edicts/insolvencies,
trade licenses, VAT/email validation, EU sanctions screening, monitoring)
from [firmafind.at](https://firmafind.at) — via skill + hosted MCP server.

## Install

```bash
# Add the marketplace (once)
/plugin marketplace add firmafind/claude-plugin

# Install the plugin
/plugin install firmafind@firmafind
```

You will be prompted for your **firmafind API key**
(`ff_live_...` from https://firmafind.at/dashboard/keys).
It is stored as a sensitive `userConfig` value and sent as the
`x-api-key` header to the hosted MCP server at
`https://firmafind.at/api/mcp`.

Update later with:

```bash
/plugin marketplace update
/plugin update firmafind@firmafind
```

## Contents

- `skills/firmafind/SKILL.md` — agent guidance: REST quick reference,
  MCP tool list, auth, credits & error handling. Mirror of
  `.agents/skills/firmafind/SKILL.md` (keep both in sync, same version).
- `.mcp.json` — hosted MCP server (`https://firmafind.at/api/mcp`,
  Streamable HTTP) using `${user_config.apiKey}`.
- `.claude-plugin/plugin.json` — plugin manifest (name, version,
  `userConfig.apiKey` prompt).

## Versioning

Plugin `version` in `.claude-plugin/plugin.json` and in
`.claude-plugin/marketplace.json` pins updates: users only receive a new
copy when the version string changes. Bump all three on release:

1. `plugins/firmafind/.claude-plugin/plugin.json` → `version`
2. `.claude-plugin/marketplace.json` → `plugins[0].version`
3. `plugins/firmafind/skills/firmafind/SKILL.md` +
   `.agents/skills/firmafind/SKILL.md` → frontmatter `version`

See `AGENTS.md` → API Change Checklist (OpenAPI / Skill / MCP /
Claude marketplace must stay in sync).
