# firmafind Claude Plugin Marketplace

Claude Code marketplace for [firmafind.at](https://firmafind.at) — structured Austrian company data (Firmenbuch/company register, financials, filings, edicts/insolvencies, trade licenses, VAT/email validation, EU sanctions screening, monitoring) via skill + hosted MCP server.

## Install

```bash
# Add the marketplace (once)
/plugin marketplace add firmafind/claude-plugin

# Install the plugin
/plugin install firmafind@firmafind
```

You will be prompted for your **firmafind API key** (`ff_live_...` from https://firmafind.at/dashboard/keys).

## Contents

- `plugins/firmafind/` — the plugin (see its [README](plugins/firmafind/README.md))
- `.claude-plugin/marketplace.json` — marketplace manifest
