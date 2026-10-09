# treg-cli

A small JSON command-line client for [treg.to](https://treg.to), a pay-per-call gateway to about 3,800 data API endpoints from about 110 vendors.

## What treg is

Treg does not collect data. It holds accounts with data vendors (TikHub, ScrapeCreators, Apollo, Hunter, Exa, DataForSEO, People Data Labs, and others) and relays your request to them. You pay from a prepaid balance, per call, at the vendor's price. There is no subscription and no vendor signup. The service is open source: https://github.com/superdesigndev/treg.

This client calls the treg HTTP API directly. It does not need the official Python CLI, and it does not install an MCP server or skills.

## Prerequisites

- Ruby 3.x (standard library only)
- A treg token (dashboard at https://treg.to, Team, API keys)

## Setup

```bash
cp .env.example .env          # then set TREG_TOKEN
ln -sf ~/tools/treg-cli/treg-cli ~/bin/treg-cli
treg-cli whoami
```

## Commands

| Command | What it does |
| --- | --- |
| `whoami` | Email, team, and role for the token |
| `balance [-n N]` | Prepaid balance and recent ledger entries |
| `platforms` | Catalog shelves with endpoint counts and providers |
| `search "job" [-n N] [--new] [--full]` | Endpoints that do a job, with price and measured success rate |
| `get <endpoint-id>` | Full endpoint record: parameters, cost, siblings |
| `call <endpoint-id or url> [-q k=v] [-d JSON] [-X M] [-H h] [--max-cost USD] [--no-cache]` | Make a call. Spend is capped at $0.10 per call unless `--max-cost` is set |
| `calls [-n N] [--days N]` | Audit log |
| `result <call-id>` | Fetch a stored answer again at no cost |
| `tools` | The team's own registered tools |
| `request "capability"` | Ask treg to add a capability |
| `raw METHOD /path [-q k=v] [-d JSON]` | Any other API path |

All output is one line of JSON on stdout: `{"ok": true, "data": ...}` or `{"ok": false, "error": ..., "code": ...}`. Exit code is 0 or 1.

## How it works

- Auth: `X-Treg-Token` header. The token includes the team, so no org header is needed.
- Calls go to `https://treg.to/call/<endpoint-id>`. Method, query, and body pass through to the vendor. The vendor's body comes back verbatim in `data.result`.
- Treg metadata comes from response headers: `X-Treg-Call-Id`, `X-Treg-Cost-Micro` (micro-USD), `X-Treg-Served-By`, `X-Treg-Cache`, `X-Treg-Async`.
- Full API notes: https://treg.to/llms.txt
