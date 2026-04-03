---
name: auto-dev
description: Use when working with Auto.dev APIs, vehicle data, VIN decoding, car listings, vehicle photos, specs, recalls, payments, interest rates, taxes, OEM build data, plate-to-VIN, CLI commands, MCP tools, or SDK methods for any automotive data task
version: 1.1.2
tags:
  - automotive
  - vehicle-data
  - vin
  - listings
  - api
  - auto-dev
  - car-search
  - recalls
  - mcp
  - sdk
  - cli
homepage: https://github.com/drivly/auto-dev-skill
metadata:
  openclaw:
    env:
      AUTODEV_API_KEY:
        description: Optional — only needed for Direct API usage. CLI and MCP use OAuth via `auto login` instead.
        required: false
        secret: true
---

# Auto.dev

Automotive data for AI agents — via MCP tools, CLI commands, SDK methods, or direct API calls.

## Detect Your Surface

Check in this order. Use the first one available:

**1. MCP** — Check if `auto_decode` is in your available tools.
If yes: use `auto_*` tools for everything. Do NOT make raw HTTP calls.
To install: `npx @auto.dev/sdk mcp install` (installs globally and configures Claude Code, Claude Desktop, Cursor).

**2. CLI** — Run `auto --version`.
If installed: authenticate once with `auto login` (OAuth — no API key needed), then use `auto` commands.
To install: `npm install -g @auto.dev/sdk` or use `npx @auto.dev/sdk <command>` without installing.

**3. SDK** — Check if `@auto.dev/sdk` is in project dependencies.
If yes: use typed SDK methods. See code-patterns.md for all methods.
To install: `npm install @auto.dev/sdk`

**4. Direct API** — Check for `AUTODEV_API_KEY` env var.
If not set: ask the user for it or direct them to https://auto.dev/pricing (free Starter plan).

## MCP Tools

If MCP tools are available, this is your only section. Use `auto_docs` to look up
parameters and response shapes before calling an unfamiliar endpoint.

```
auto_docs listings      # see all listing filters
auto_docs payments      # see payment params
auto_docs vin-decode    # see VIN decode response shape
```

| Tool | Description | Plan |
|------|-------------|------|
| `auto_decode` | Decode a VIN | Starter |
| `auto_listings` | Search listings with filters | Starter |
| `auto_photos` | Get vehicle photos | Starter |
| `auto_specs` | Vehicle specifications | Growth |
| `auto_build` | OEM build data ($0.10/call) | Growth |
| `auto_recalls` | Safety recalls | Growth |
| `auto_payments` | Payment calculations | Growth |
| `auto_apr` | Interest rates | Growth |
| `auto_tco` | Total cost of ownership | Growth |
| `auto_open_recalls` | Open/unresolved recalls | Scale |
| `auto_plate` | Plate to VIN ($0.55/call) | Scale |
| `auto_taxes` | Taxes and fees | Scale |
| `auto_docs` | Search bundled API documentation | — |
| `auto_config_set` | Set a config value (e.g. `raw: true`) | — |
| `auto_config_get` | Get a config value or list all settings | — |

API metadata is stripped from MCP tool responses by default. Use `auto_config_set` with `key: "raw"` and `value: "true"` to get full responses.

**If MCP tools are available, skip to Important Rules below.**

## CLI Quick Reference

Install: `npm install -g @auto.dev/sdk` (or use `npx @auto.dev/sdk` without installing)
Authenticate once: `auto login`

```
auto decode <vin>
auto photos <vin>
auto listings --make Toyota --year 2024 --price 10000-40000 --state CA
auto specs <vin>
auto build <vin>
auto recalls <vin>
auto open-recalls <vin>
auto payments <vin> --price 35000 --zip 90210 --down-payment 5000
auto apr <vin> --year 2024 --make Honda --model Accord --zip 90210 --credit-score 750
auto tco <vin> --zip 90210
auto taxes <vin> --price 35000 --zip 90210
auto plate <state> <plate>
auto usage
auto docs [query]        # search bundled docs
auto explore [endpoint]  # browse params and mappings
```

All commands support `--json`, `--yaml`, `--raw`, and `--api-key <key>` flags.

API metadata is stripped by default — you get clean vehicle data. Use `--raw` to see the full API response.

```
auto config set raw true    # always show full responses
auto config get raw         # check current value
auto config list            # show all settings
```
Full CLI reference: https://docs.auto.dev/v2/cli-mcp-sdk

**If CLI is available, skip to Important Rules below.**

## Direct API Quick Reference

Use these only when MCP and CLI are unavailable.

**V2** (base: `https://api.auto.dev`): `Authorization: Bearer {key}` or `?apiKey={key}`
**V1** (base: `https://auto.dev/api`): `?apikey={key}` (query string only)

| Endpoint | Plan | Required Params |
|----------|------|-----------------|
| `GET /listings?filters` | Starter | See v2-listings-api.md |
| `GET /listings/{vin}` | Starter | — |
| `GET /vin/{vin}` | Starter | — |
| `GET /photos/{vin}` | Starter | — |
| `GET /specs/{vin}` | Growth | — |
| `GET /build/{vin}` | Growth | — ($0.10/call) |
| `GET /recalls/{vin}` | Growth | — |
| `GET /tco/{vin}` | Growth | — |
| `GET /payments/{vin}` | Growth | `price`, `zip` |
| `GET /apr/{vin}` | Growth | `year`, `make`, `model`, `zip`, `creditScore` |
| `GET /openrecalls/{vin}` | Scale | — |
| `GET /plate/{state}/{plate}` | Scale | — ($0.55/call) |
| `GET /taxes/{vin}` | Scale | — |

V1 supplemental (no V2 equivalent): `/api/models`, `/api/cities`, `/api/zip/{zip}`, `/api/autosuggest/{term}`

Before making any Direct API call, read the relevant reference file for full parameter names and examples.
Full parameter reference: v2-listings-api.md | v2-vin-apis.md | v2-plate-api.md | v1-apis.md

### Common Gotchas (All Surfaces)

These parameter names do NOT exist on the Direct API — agents hallucinate them frequently:

```
make=    ← use vehicle.make
model=   ← use vehicle.model
state=   ← use retailListing.state
price=   ← use retailListing.price (on /listings), or price= (on /payments, /taxes)
rows=    ← use limit
sort=    ← no sort parameter exists
_order=  ← no sort parameter exists
```

MCP and CLI handle these mappings automatically. This only matters for Direct API calls.

## Plans & Pricing

See pricing.md for full per-call costs and upgrade links.

| Plan | Monthly | Includes |
|------|---------|----------|
| Starter | Free + data fees | VIN Decode, Listings, Photos (1,000 free calls/mo) |
| Growth | $299/mo + data fees | + Specs, Recalls, TCO, Payments, APR, Build |
| Scale | $599/mo + data fees | + Open Recalls, Plate-to-VIN, Taxes & Fees |

All plans charge per-call data fees on every request. Growth/Scale have no cap on volume but data fees still apply.

## Important Rules

- **Before batch operations**, call the endpoint once with a single item to see the actual response shape. Do not guess field names or nesting — inspect the real response first.
- **Include ALL fields** from the API response unless the user explicitly says to exclude some.
- **Small results** (<10 items, single VIN): Display inline as formatted table
- **Large results** (10+ listings): Ask user preference, default to CSV export
- **Always support**: CSV, JSON export when user requests
- **Chain APIs** when the query spans multiple endpoints — MCP tools and CLI commands can be called in parallel

## Deep Reference

**MCP or CLI:** use `auto_docs [query]` or `auto docs [query]` for live parameter lookup.
Full reference: https://docs.auto.dev/v2/cli-mcp-sdk

**SDK:** see code-patterns.md for typed methods and response format.

**Direct API:**
- API Docs: v2-listings-api.md | v2-vin-apis.md | v2-plate-api.md | v1-apis.md
- Workflows: chaining-patterns.md | interactive-explorer.md | business-workflows.md
- Code Gen: code-patterns.md | app-scaffolding.md | integration-recipes.md
- Other: error-recovery.md | pricing.md | examples.md
