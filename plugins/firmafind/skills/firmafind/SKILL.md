---
name: firmafind
description: |
 Unified API for Austrian companies. Search, verify and retrieve structured company data from Austrian register sources through one API. Company register data, available filings, VAT validation, email validation and insolvency notices. One API key. One JSON format. Built for product workflows, internal tools and AI agents.
license: FSL-1.1-Apache-2.0
version: "2.5.0"
---

# firmafind Agent Skill

This skill guides AI agents in querying Austrian business register data, retrieving corporate records, financial statements, and connecting to the firmafind Hosted MCP Server or REST API.

## Overview

firmafind converts Austrian Firmenbuch and Edikte data into structured JSON APIs and an agent-ready Hosted Model Context Protocol (MCP) server.

Unified API for Austrian companies. Search, verify and retrieve structured company data from Austrian register sources through one API. Company register data, available filings, VAT validation, email validation, insolvency notices, and company monitoring. One API key. One JSON format.

Target domain:

- **Company Registration Numbers (Firmenbuchnummer / `fnr`)**: Numbers followed by a single lowercase letter, e.g. `123456x` or `fn123456x`.
- **Legal Forms**: `GmbH` (Gesellschaft mit beschränkter Haftung), `AG` (Aktiengesellschaft), `KG`, `OG`, `e.U.` (eingetragenes Einzelunternehmen).
- **Data Points**: Management structure (Geschäftsführer, Prokuristen), share capital, registered seat, annual financial reports (Jahresabschlüsse), insolvency notices (Edikte), EU financial sanctions screening, and company change monitoring.

## Authentication

All API and MCP requests require an API key in the `x-api-key` HTTP header, `Authorization: Bearer` header, or `?apiKey=` query parameter.

```http
GET /api/companies?name=Example HTTP/1.1
Host: firmafind.at
x-api-key: ff_live_your_key_here
```

API keys can be managed in the dashboard at https://firmafind.at/dashboard/keys.

## Hosted MCP server integration (`/api/mcp`)

Connect Claude Desktop, Cursor, OpenCode, or Windsurf directly to the hosted server. Requires a paid subscription (Starter, Pro or Business) or active trial (free plan is REST only):

- **Server URL**: `https://firmafind.at/api/mcp`
- **Transport**: Web Standards Streamable HTTP / Server-Sent Events (SSE)
- **Header**: `x-api-key: YOUR_API_KEY`

### Available MCP tools

1. `search_austrian_company`: Search Firmenbuch by name, court, or legal form.
2. `get_company_details`: Complete Firmenbuch registration extract (address, capital, directors, shareholders).
3. `get_trade_licenses`: Search Austrian trade licenses by Firmenbuchnummer, company name, or holder surname.
4. `get_company_changes`: Historical register changes and new formations across a date range.
5. `list_company_documents`: List available annual reports, balance sheets, and filed documents.
6. `download_company_document`: Download document contents & structured annual financial report XML parsed as JSON (costs 10 credits).
7. `get_company_financials`: Normalized published annual financial statements (up to 5 periods, metrics, ratios, changes, quality + evidence). Costs 15 credits for years=1, 25 for years=2..5.
8. `list_edikte`: Search Austrian court edicts for corporate insolvencies.
9. `get_edict_details`: Publication details for an insolvency or court notice.
10. `validate_email`: Validate email deliverability (syntax, MX/DNS, disposable, role-based, typo suggestions).
11. `screen_eu_sanctions`: Screen entities or individuals against the EU Consolidated Sanctions List.
12. `list_monitors`: List all company monitors (watchlists), watched companies, and plan quota limits.
13. `watch_company`: Add an Austrian company (by Firmenbuchnummer) to a monitoring watchlist.
14. `unwatch_company`: Remove an Austrian company from a monitoring watchlist.
15. `list_monitor_events`: Retrieve timeline events (filings, status changes, insolvency notices) for a monitor.
16. `list_monitor_webhooks`: List webhook push subscriptions for a monitor.
17. `create_monitor_webhook`: Subscribe an HTTPS endpoint to receive signed real-time business events (HMAC-SHA256, optional event type filter).
18. `delete_monitor_webhook`: Remove a webhook subscription from a monitor.
19. `test_monitor_webhook`: Dispatch a signed test event to verify endpoint connectivity.

## REST API quick reference

- **Base URL**: `https://firmafind.at`
- **Auth Header**: `x-api-key: YOUR_API_KEY`
- **MCP URL**: `https://firmafind.at/api/mcp`

| Endpoint | Method | Purpose |
| -------------------------------- | ------ | ------------------------------------------- |
| `/api/companies/search` | GET | Search companies by name, legal form, court |
| `/api/companies/{fnr}` | GET | Full Firmenbuch company details by Firmenbuchnummer |
| `/api/trade` | GET | Search trade licenses by `fnr`, `name`, or `familienname` (+ optional `vorname`; alias `/api/trade/search`) |
| `/api/company/changes` | GET | Company register changes ticker |
| `/api/documents` | GET | List available filings for a company |
| `/api/documents/{key}` | GET | Download document (PDF/XML + parsed JSON) |
| `/api/financials/{fnr}` | GET | Normalized financials: `?years=1..5&include=summary,periods,changes,evidence,documents` |
| `/api/edikte` | GET | Search insolvency edicts |
| `/api/edikte/{type}/{id}` | GET | Edikt details |
| `/api/vat/validate` | GET | VAT validation via VIES |
| `/api/email/validate` | GET/POST | Email validation & deliverability |
| `/api/sanctions/search` | GET | EU sanctions screening |
| `/api/monitors` | GET/POST | List and create company monitors |
| `/api/monitors/{id}` | GET/DELETE | Monitor details with companies, or delete monitor |
| `/api/monitors/{id}/companies` | POST | Add companies to monitor (bulk) |
| `/api/monitors/{id}/companies/{fnr}` | DELETE | Remove company from monitor |
| `/api/monitors/{id}/events` | GET | Monitor event stream & timeline with cursor pagination |
| `/api/monitors/{id}/subscriptions` | GET/POST | List and create webhook push subscriptions |
| `/api/monitors/{id}/subscriptions/{subId}` | DELETE | Delete webhook subscription |
| `/api/monitors/{id}/subscriptions/{subId}/test` | POST | Send signed test webhook event |

## Error handling and credits

- Standard lookups cost **1 credit**.
- Annual financial report / document downloads cost **10 credits**.
- Financials API (`GET /api/financials/{fnr}`) costs **15 credits** for `years=1`, **25 credits** for `years=2..5`. Validation errors, 401, 402, 404, 429, upstream (502) and parser (503) failures cost **0 credits**.
- Financials values come from published filings (XML primary source; PDF never parsed in v1). Every metric has an availability state (`reported`, `not_disclosed`, `not_mapped`, `not_available_in_source_format`, `parse_failed`); `null` never means zero. PDF-only periods are returned as placeholders with null metrics so `summary` counts agree — check `latestStructuredPeriodEnd` vs `latestPeriodEnd`, `source.filingScope`, and the `newer_filing_available_without_structured_extraction` observation. `employees` is always `not_mapped` in v1. Not legal, accounting, tax, investment, credit or compliance advice.
- HTTP 401: Invalid or missing API key.
- HTTP 402 / 429: Credit balance exhausted or quota reached.
