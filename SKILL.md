---
name: auto-dev
description: Use when working with Auto.dev APIs, vehicle data, VIN decoding, car listings, vehicle photos, specs, recalls, payments, interest rates, taxes, OEM build data, plate-to-VIN, or any automotive data API calls
---

# Auto.dev API

Comprehensive automotive data platform. V2 is the primary API. V1 provides supplemental endpoints with no V2 equivalent.

## Authentication

Check for `AUTODEV_API_KEY` environment variable first. If not set, ask the user for their API key.

**V2** (base: `https://api.auto.dev`): `Authorization: Bearer {key}` or `?apiKey={key}`
**V1** (base: `https://auto.dev/api`): `?apikey={key}` (query string only)

## V2 API — Quick Reference (Primary)

| Endpoint | Plan | Use When |
|----------|------|----------|
| `GET /listings` | Starter | Search inventory by make/model/price/location |
| `GET /listings/{vin}` | Starter | Get single listing by VIN |
| `GET /vin/{vin}` | Starter | Decode VIN to make/model/trim/origin |
| `GET /photos/{vin}` | Starter | Get vehicle photos |
| `GET /specs/{vin}` | Growth | Full specs, colors, engine, dimensions |
| `GET /build/{vin}` | Growth | Factory options, OEM colors, option MSRP |
| `GET /recalls/{vin}` | Growth | All NHTSA recalls for a VIN |
| `GET /tco/{vin}` | Growth | 5-year total cost of ownership |
| `GET /payments/{vin}` | Growth | Monthly payment calculator |
| `GET /apr/{vin}` | Growth | Interest rates by credit score/term |
| `GET /openrecalls/{vin}` | Scale | Only unresolved recalls |
| `GET /plate/{state}/{plate}` | Scale | License plate to VIN |
| `GET /taxes/{vin}` | Scale | State/local taxes and fees |

## V1 API — Supplemental (No V2 Equivalent)

| Endpoint | Use When |
|----------|----------|
| `GET /models` | List all available makes/models |
| `GET /cities` | City data by state |
| `GET /zip/{zip}` | ZIP to coordinates/city/DMA |
| `GET /autosuggest/{term}` | Type-ahead for makes/models |

Default to V2. Use V1 only for the 4 endpoints above that have no V2 equivalent.

## Plans & Pricing

See pricing.md for full per-call costs and upgrade links. Quick summary:

| Plan | Monthly | Includes |
|------|---------|----------|
| Starter | Free + data fees | VIN Decode, Listings, Photos (1,000 free calls/mo) |
| Growth | $299/mo + data fees | + Specs, Recalls, TCO, Payments, APR, Build |
| Scale | $599/mo + data fees | + Open Recalls, Plate-to-VIN, Taxes & Fees |

All plans charge per-call data fees on every request. Growth/Scale have no cap on volume (never throttled or blocked), but data fees still apply. Scale has lower per-call costs than Growth. See pricing.md for exact costs and upgrade links.

## Output Patterns

- **Small results** (<10 items, single VIN): Display inline as formatted table
- **Large results** (10+ listings): Ask user preference, default to CSV export
- **Always support**: CSV, JSON export when user requests
- **Chain APIs** when the query spans multiple endpoints

## Deep Reference

**API Docs:** v2-listings-api.md | v2-vin-apis.md | v2-plate-api.md | v1-apis.md
**Workflows:** chaining-patterns.md | interactive-explorer.md | business-workflows.md
**Code Gen:** code-patterns.md | app-scaffolding.md | integration-recipes.md
**Other:** error-recovery.md | pricing.md | examples.md
