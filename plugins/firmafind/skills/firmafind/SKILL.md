---
name: firmafind
description: |
 Unified API for Austrian companies. Search, verify and retrieve structured company data from Austrian register sources through one API. Company register data, available filings, VAT validation, email validation and insolvency notices. One API key. One JSON format. Built for product workflows, internal tools and AI agents.
license: MIT
version: "2.9.0"
---

# firmafind Agent Skill

This skill guides AI agents in querying Austrian business register data, retrieving corporate records, financial statements, and connecting to the firmafind Hosted MCP Server or REST API.

## Overview

firmafind converts Austrian Firmenbuch and Edikte data into structured JSON APIs and an agent-ready hosted MCP server.

Unified API for Austrian companies. Search, verify and retrieve structured company data from Austrian register sources through one API. Company register data, available filings, VAT validation, email validation, insolvency notices, and company monitoring. One API key. One JSON format.

Target domain:

- **Company Registration Numbers (Firmenbuchnummer / `fnr`)**: Numbers followed by a single lowercase letter, e.g. `123456x` or `fn123456x`.
- **Legal Forms**: `GmbH` (Gesellschaft mit beschränkter Haftung), `AG` (Aktiengesellschaft), `KG`, `OG`, `e.U.` (eingetragenes Einzelunternehmen).
- **Data Points**: Management structure (Geschäftsführer, Prokuristen), share capital, registered seat, annual financial reports (Jahresabschlüsse), insolvency notices (Edikte), and company change monitoring.
- **Monitoring Modes**:
  - **Watchlist Monitoring (`/api/monitors`)**: Monitor specific companies by Firmenbuchnummer for balance sheet filings, management events, seat/address changes, and insolvency notices.
  - **New Company Feed (`/api/monitors/new-companies/subscriptions`)**: Webhook stream for newly registered Austrian companies with optional filtering by legal form (`GmbH`, `FlexKapG`, `e.U.`), registration court, or state (up to 2 webhook URLs per account).

## Authentication

Customer API and MCP requests require an API key in the `x-api-key` HTTP header, `Authorization: Bearer` header, or `?apiKey=` query parameter.

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

1. `search_austrian_company`: Search Firmenbuch by name, court, or legal form, including official industry.
2. `get_company_details`: Complete Firmenbuch registration extract (address, capital, directors, shareholders), including official industry.
3. `get_trade_licenses`: Search Austrian trade licenses by Firmenbuchnummer, company name, or holder surname.
4. `get_company_changes`: Historical register changes and new formations across a date range.
5. `list_company_documents`: List available annual reports, balance sheets, and filed documents.
6. `download_company_document`: Download document contents & structured annual financial report XML parsed as JSON (costs 10 credits).
7. `get_company_financials`: Normalized published annual financial statements (up to 5 periods, metrics, ratios, changes, quality + evidence). Costs 15 credits for years=1, 25 for years=2..5.
8. `list_edikte`: Search Austrian court edicts for corporate insolvencies (companies only, no personal data).
9. `get_edict_details`: Publication details for a company insolvency or court notice (companies only, no personal data).
10. `validate_email`: Validate email deliverability (syntax, MX/DNS, disposable, role-based, typo suggestions).
11. `list_monitors`: List all company monitors (watchlists), watched companies, and plan quota limits.
12. `watch_company`: Add an Austrian company (by Firmenbuchnummer) to a monitoring watchlist.
13. `unwatch_company`: Remove an Austrian company from a monitoring watchlist.
14. `list_monitor_events`: Retrieve timeline events (filings, status changes, insolvency notices) for a monitor.
15. `list_monitor_webhooks`: List webhook push subscriptions for a monitor.
16. `create_monitor_webhook`: Subscribe an HTTPS endpoint to receive signed business events (HMAC-SHA256, optional event type filter).
17. `delete_monitor_webhook`: Remove a webhook subscription from a monitor.
18. `test_monitor_webhook`: Dispatch a signed test event to verify endpoint connectivity.
19. `list_new_company_subscriptions`: List all active new company registration webhook subscriptions.
20. `create_new_company_subscription`: Subscribe an HTTPS webhook to receive batch notifications for newly registered Austrian companies (optional filters by legal form, court, state).
21. `delete_new_company_subscription`: Remove a new company registration webhook subscription.

22. `discover_companies` (private pilot only): Paginated company discovery by ÖNACE, trade evidence, region and registration date.
23. `get_discovered_company` (private pilot only): Indexed company and all classification evidence.

Tools 22–23 are absent when `FEATURE_INDUSTRY_DISCOVERY=false`; when enabled they also require the user’s existing `industry_discovery` rollout access. Do not call or probe pilot endpoints unless the operator has confirmed that access is enabled.

## REST API quick reference

- **Base URL**: `https://firmafind.at`
- **Auth Header**: `x-api-key: YOUR_API_KEY`
- **MCP URL**: `https://firmafind.at/api/mcp`

| Endpoint | Method | Purpose |
| -------------------------------- | ------ | ------------------------------------------- |
| `/api/companies/search` | GET | Search companies by name, legal form, court; official industry |
| `/api/companies/{fnr}` | GET | Full Firmenbuch company details and official industry by Firmenbuchnummer |
| `/api/trade` | GET | Search trade licenses by `fnr`, `name`, or `familienname` (+ optional `vorname`; alias `/api/trade/search`) |
| `/api/company/changes` | GET | Company register changes ticker |
| `/api/documents` | GET | List available filings for a company |
| `/api/documents/{key}` | GET | Download document (PDF/XML + parsed JSON) |
| `/api/financials/{fnr}` | GET | Normalized financials: `?years=1..5&include=summary,periods,changes,evidence,documents` |
| `/api/edikte` | GET | Search company insolvency edicts (no personal data) |
| `/api/edikte/{type}/{id}` | GET | Company edikt details (no personal data) |
| `/api/vat/validate` | GET | VAT validation via VIES |
| `/api/email/validate` | GET/POST | Email validation & deliverability |
| `/api/monitors` | GET/POST | List and create company monitors |
| `/api/monitors/{id}` | GET/PATCH/DELETE | Monitor details with companies, update metadata, or delete monitor |
| `/api/monitors/{id}/companies` | GET/POST | List monitored companies (paginated + search) or add companies (bulk) |
| `/api/monitors/{id}/companies/{fnr}` | DELETE | Remove company from monitor |
| `/api/monitors/{id}/events` | GET | Monitor event stream & timeline with cursor pagination |
| `/api/monitors/{id}/subscriptions` | GET/POST | List and create webhook push subscriptions |
| `/api/monitors/{id}/subscriptions/{subId}` | PATCH/DELETE | Update webhook subscription (URL, filters, secret) or delete |
| `/api/monitors/{id}/subscriptions/{subId}/test` | POST | Send signed test webhook event |
| `/api/monitors/new-companies/subscriptions` | GET/POST | List and create new company registration webhooks |
| `/api/monitors/new-companies/subscriptions/{id}` | PATCH/DELETE | Update or delete new company webhook |
| `/api/monitors/new-companies/subscriptions/{id}/test` | POST | Send signed test new company webhook event |

## Error handling and credits

- Standard lookups cost **1 credit**.
- Annual financial report / document downloads cost **10 credits**.
- Financials API (`GET /api/financials/{fnr}`) costs **15 credits** for `years=1`, **25 credits** for `years=2..5`. Validation errors, 401, 402, 404, 429, upstream (502) and parser (503) failures cost **0 credits**.
- Financials values come from published filings (XML primary source; PDF never parsed in v1). Every metric has an availability state (`reported`, `not_disclosed`, `not_mapped`, `not_available_in_source_format`, `parse_failed`); `null` never means zero. PDF-only periods are returned as placeholders with null metrics so `summary` counts agree — check `latestStructuredPeriodEnd` vs `latestPeriodEnd`, `source.filingScope`, and the `newer_filing_available_without_structured_extraction` observation. `employees` is always `not_mapped` in v1. Not legal, accounting, tax, investment, credit or compliance advice.
- HTTP 401: Invalid or missing API key.
- HTTP 402 / 429: Credit balance exhausted or quota reached.

## Official ÖNACE 2025 enrichment

Company search, details, public search and their existing MCP tools return optional `industry`, or `null` when unavailable. Example (timestamp illustrative):

```json
{"industry":{"code":"01110","labelDe":"Anbau von anderem Getreide als Reis sowie von Hülsenfrüchten und Ölsaaten","classificationVersion":"2025","source":"Statistik Austria - data.statistik.gv.at","updatedAt":"2026-10-24T09:00:00.000Z"}}
```

Codes are five-digit official subclasses; preserve leading zeros. No classifications are inferred. `updatedAt` is the successful snapshot import time, not the date of a company activity change.

Datenquelle: Statistik Austria – data.statistik.gv.at. [Mappings](https://data.statistik.gv.at/data/OGDEXT_HVD_FBNR_OENACE_1.csv) and [classification](https://data.statistik.gv.at/data/OGDEXT_KLASSDB_OENACE2025_1.zip). [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). Firmafind normalizes Firmenbuchnummern and joins codes with German labels. Retain attribution and source/license links when redistributing, indicate modifications, and do not imply endorsement.

## Private industry-discovery pilot

Disabled by default. The server-only `FEATURE_INDUSTRY_DISCOVERY` master switch and the existing per-user `industry_discovery` flag must both allow access. Disabled REST endpoints return 404 before authentication or billing; the discovery page remains unavailable and no public documentation links appear. Dashboard navigation remains visible but disabled for accounts without access. Existing API endpoints retain their behavior.

| Private endpoint | Method | Purpose |
| --- | --- | --- |
| `/api/discovery/companies` | GET | Search indexed registrations using `from`, `to`, `state`, `postalCode`, `oenace` (2–5 digit prefix), `industry`, `area`, `mode`, `limit`, `cursor` |
| `/api/discovery/companies/{fnr}` | GET | Retrieve one indexed company with every stored classification/evidence record |

Example pilot response fragment (synthetic dates and name):

```json
{"data":[{"fnr":"123456x","name":"Example GmbH","registeredAt":"2026-10-01","state":"Wien","classificationPending":true,"matches":[{"industryId":"construction.tiles.installation","official":false,"classificationVersion":"firmafind-trades-v1","source":"firmenbuch_business","sourceRef":"licensed business extract","permissionRef":"operator permission reference","evidence":"Verlegung von Fliesen","confidence":0.75,"status":"probable","ruleVersion":"2026-10-24.1","observedAt":"2026-10-24T10:00:00.000Z","firstSeenAt":"2026-10-24T10:00:00.000Z"}]}],"meta":{"count":1,"nextCursor":null}}
```

Official ÖNACE matches have `official=true` and version `2025`. Broad official codes never prove a specific trade. Trade matches are inferred even when based on official GISA wording; deterministic scores describe rules, not calibrated probabilities. `strict` requires >=0.85, `balanced` >=0.60, `inclusive` >=0.30. A company may have many matches. Unknown classifications/regions remain missing. Original registration and later evidence observation times are distinct. Successful REST and MCP lookups cost one existing API credit.

Current discovery jobs do not call GISA: its published terms exclude industry/founder-list retrieval, and no discovery-specific permission was found. Only import evidence acquired under a permission covering the intended purpose, retaining the permission reference. No industry webhook subscriptions or deliveries are implemented in this iteration.
