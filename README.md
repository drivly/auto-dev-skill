<p align="center">
  <img src="assets/autodev.png" alt="Auto.dev" />
</p>

<h1 align="center">Auto.dev Agent Skill</h1>

<p align="center">
  <a href="https://github.com/drivly/auto-dev-skill/blob/main/LICENSE"><img src="https://img.shields.io/github/license/drivly/auto-dev-skill" alt="License" /></a>
  <a href="https://github.com/drivly/auto-dev-skill/stargazers"><img src="https://img.shields.io/github/stars/drivly/auto-dev-skill" alt="Stars" /></a>
  <a href="https://github.com/drivly/auto-dev-skill/issues"><img src="https://img.shields.io/github/issues/drivly/auto-dev-skill" alt="Issues" /></a>
  <a href="https://skills.sh/drivly/auto-dev-skill/auto-dev"><img src="https://img.shields.io/badge/skills.sh-auto--dev-blue" alt="Skills.sh" /></a>
  <a href="https://docs.auto.dev/v2/cli-mcp-sdk"><img src="https://img.shields.io/badge/docs-auto.dev-black" alt="Docs" /></a>
  <a href="https://clawhub.ai/bryant22/auto-dev"><img src="https://img.shields.io/badge/clawhub-auto--dev-orange" alt="ClawHub" /></a>
  <a href="https://glama.ai/mcp/servers/drivly/auto-dev-skill"><img src="https://glama.ai/mcp/servers/drivly/auto-dev-skill/badges/score.svg" alt="Glama Score" /></a>
</p>

<p align="center">
Give any AI coding agent superpowers with <a href="https://auto.dev">Auto.dev</a> automotive data APIs. Search vehicle listings, decode VINs, calculate payments, check recalls, and more — all through natural conversation.
</p>

<p align="center">
Works with <strong>Claude Code</strong>, <strong>Cursor</strong>, <strong>Codex</strong>, <strong>GitHub Copilot</strong>, <strong>Windsurf</strong>, <strong>OpenClaw</strong>, and <a href="https://github.com/vercel-labs/skills#supported-agents">40+ other agents</a>.
</p>

## Install

### CLI + MCP + SDK (one command)

```bash
npm install -g @auto.dev/sdk
```

This gives you the `auto` CLI, the MCP server for AI agents, and the JS/TypeScript SDK.
Full reference: [docs.auto.dev/v2/cli-mcp-sdk](https://docs.auto.dev/v2/cli-mcp-sdk)

### Configure MCP for your AI tools

```bash
auto mcp install
```

Auto-configures the MCP server in Claude Code, Claude Desktop, and Cursor in one step.

### Authenticate

```bash
auto login
```

OAuth login — no API key needed for CLI and MCP usage after this.

### Coding agent skill (Claude Code, Cursor, Codex, Windsurf)

```bash
npx skills add drivly/auto-dev-skill
```

```bash
# Install globally
npx skills add drivly/auto-dev-skill -g

# Install to a specific agent
npx skills add drivly/auto-dev-skill -a claude-code

# Install to multiple agents
npx skills add drivly/auto-dev-skill -a claude-code -a cursor -a codex
```

### OpenClaw / ClawHub

```bash
clawhub install auto-dev
```

Or paste the URL directly into your OpenClaw chat:

```
Install this skill: https://github.com/drivly/auto-dev-skill
```

### Set Your API Key

If you're using the CLI or MCP after `auto login`, you don't need an API key.

For direct API usage, sign up at [auto.dev](https://auto.dev) and get your key
from the [dashboard](https://auto.dev/dashboard):

```bash
export AUTODEV_API_KEY="sk_ad_your_key_here"
```

Add this to your shell profile (`~/.zshrc`, `~/.bashrc`) to persist it.

## What It Does

Instead of reading API docs and crafting curl commands, just describe what you need:

- **"Find all Toyota RAV4s under $30k near Miami"** — searches listings with the right filters
- **"Decode this VIN: JM3KKAHD5T1379650"** — calls VIN decode, returns structured data
- **"Decode these VINs and enrich the CSV"** — batch decodes a CSV of VINs and adds year, make, model, trim, engine, drivetrain, and every other field
- **"Validate all of these VINs to make sure they're real"** — checks VIN format, checksum, and decode status to flag invalid or fake VINs
- **"Compare these two cars side by side"** — chains specs, payments, and TCO endpoints in parallel
- **"Export all Honda Civics in Texas to CSV"** — paginates through results and saves to file
- **"What's the monthly payment on this car with $10k down?"** — calculates financing with real rates
- **"Check if this car has any open recalls"** — safety check before you buy
- **"Build me a car search app with Next.js"** — scaffolds a full project wired to Auto.dev APIs

The skill handles authentication, pagination, error handling, cost estimation, and output formatting automatically.

## Supported APIs

### V2 APIs (Primary)

| Endpoint | Plan | Description |
|----------|------|-------------|
| Vehicle Listings | Starter | Search millions of active listings |
| VIN Decode | Starter | Decode any VIN to vehicle data |
| Vehicle Photos | Starter | High-quality vehicle images |
| Specifications | Growth | Full technical specs and features |
| OEM Build Data | Growth | Factory options, colors, MSRP |
| Vehicle Recalls | Growth | NHTSA recall history |
| Total Cost of Ownership | Growth | 5-year ownership cost analysis |
| Vehicle Payments | Growth | Loan payment calculator |
| Interest Rates | Growth | APR by credit score and term |
| Open Recalls | Scale | Unresolved recalls only |
| Plate-to-VIN | Scale | License plate lookup |
| Taxes & Fees | Scale | State/local tax calculations |

### V1 APIs (Supplemental — No V2 Equivalent)

| Endpoint | Description |
|----------|-------------|
| Models | List all makes/models in database |
| Cities | US city data by state |
| ZIP Lookup | ZIP to coordinates/DMA |
| Autosuggest | Type-ahead for makes/models |

## Pricing

| Plan | Monthly | Annual | Rate Limit | What You Get |
|------|---------|--------|------------|--------------|
| **Starter** | Free + data fees | — | 5 req/s | VIN Decode, Listings, Photos (1,000 free calls/mo) |
| **Growth** | $299/mo + data fees | $249/mo | 10 req/s | + Specs, Recalls, TCO, Payments, APR, Build |
| **Scale** | $599/mo + data fees | $499/mo | 50 req/s | + Open Recalls, Plate-to-VIN, Taxes & Fees |

All plans charge per-call data fees on every request. Growth and Scale have no cap on call volume — requests are never throttled or blocked — but data fees still apply. See [auto.dev/pricing](https://www.auto.dev/pricing) for per-call costs.

If you hit an endpoint outside your plan, your agent will let you know what's needed and link you directly to upgrade.

## Examples

**Search and export:**
> "Find all Mazda CX-90s under $60k in Florida and save to CSV"

**Full vehicle report:**
> "Tell me everything about VIN WP0AA2A92PS206106"

**Payment comparison:**
> "Find SUVs under $40k near 33132 and show monthly payments with $5k down"

**Market analysis:**
> "What's the going rate for a 2023 BMW X5?"

**Build a complete app:**
> "Build me a used car search app with Next.js"

**Dealer competitive analysis:**
> "Compare pricing across Toyota dealers within 50 miles of Orlando"

**Interactive exploration:**
> "Show me SUVs under $50k in FL" → "Only AWD" → "Any recalls?" → "Payments on top 5?"

## Skill Contents

| File | Purpose |
|------|---------|
| `SKILL.md` | Entry point: auth, quick reference, plan summary |
| `pricing.md` | Full per-call costs, plan tiers, upgrade links |
| `examples.md` | Real API responses and agent output examples |
| `v2-listings-api.md` | V2 Listings: filters, pagination, response schema |
| `v2-vin-apis.md` | V2 VIN-based: decode, specs, build, photos, recalls, payments, APR, taxes, TCO |
| `v2-plate-api.md` | V2 Plate-to-VIN reference |
| `v1-apis.md` | V1 supplemental: models, cities, ZIP, autosuggest |
| `chaining-patterns.md` | Multi-endpoint composition workflows |
| `code-patterns.md` | Framework code gen (Next.js, React, Express, Flask, Python) |
| `error-recovery.md` | Input validation, model name normalization, error handling |
| `interactive-explorer.md` | Conversational search refinement across messages |
| `integration-recipes.md` | Slack, email, cron, Zapier, Google Sheets integrations |
| `business-workflows.md` | Dealer analysis, market pricing, fleet procurement, due diligence |
| `app-scaffolding.md` | Full app templates: car search, dealer dashboard, VIN lookup |

## Manage Your Skill

**Coding agent skill:**

```bash
npx skills check          # check for updates
npx skills update         # update to latest
npx skills remove auto-dev  # remove
```

**OpenClaw / ClawHub:**

```bash
clawhub update auto-dev   # update to latest
clawhub uninstall auto-dev  # remove
clawhub list              # list all installed skills
```

**CLI / MCP / SDK:**

```bash
npm update -g @auto.dev/sdk  # update everything
auto --version               # check current version
auto mcp status              # check MCP server status
```

## Contributing

We welcome contributions! Whether it's fixing an API field, adding a new workflow pattern, or supporting a new framework — see [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

## Documentation

- [Auto.dev API Docs](https://docs.auto.dev/)
- [Auto.dev Pricing](https://www.auto.dev/pricing)
- [Auto.dev Dashboard](https://auto.dev/dashboard)

## License

MIT — see [LICENSE](LICENSE) for details.

---

<p align="center">
  If this skill saves you time, <a href="https://github.com/drivly/auto-dev-skill">give it a star</a> — it helps other developers find it.
</p>
