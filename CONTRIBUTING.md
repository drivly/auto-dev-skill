# Contributing to Auto.dev Agent Skill

Thanks for your interest in contributing! This skill helps developers across 42+ AI coding agents work with Auto.dev APIs. Your contributions make it better for everyone.

## How to Contribute

### Reporting Issues

- **API changes** — If an Auto.dev endpoint changed its response format, parameters, or pricing, [open an issue](https://github.com/drivly/auto-dev-skill/issues/new?template=bug_report.md)
- **Skill bugs** — If the skill gives incorrect guidance or generates broken code, report it
- **Feature requests** — Want a new workflow pattern, framework integration, or business workflow? [Request it](https://github.com/drivly/auto-dev-skill/issues/new?template=feature_request.md)

### Submitting Changes

1. **Fork the repo** and create a branch from `main`
2. **Make your changes** — edit the relevant `.md` files
3. **Verify accuracy** — test any API calls you document against the live API
4. **Keep it concise** — the SKILL.md entry point should stay under 500 words
5. **Submit a PR** with a clear description of what changed and why

### What to Contribute

**High-value contributions:**
- New chaining patterns or business workflows
- Framework integrations (Vue, Svelte, Django, Rails, etc.)
- Error recovery entries for edge cases you've encountered
- Model name normalizations you've discovered
- Pricing updates when Auto.dev changes their plans

**File guide:**
| If you want to... | Edit this file |
|-------------------|---------------|
| Fix API parameters or response fields | `v2-listings-api.md`, `v2-vin-apis.md`, `v2-plate-api.md`, `v1-apis.md` |
| Add a new multi-endpoint workflow | `chaining-patterns.md` |
| Add a framework integration pattern | `code-patterns.md` |
| Add a business use case | `business-workflows.md` |
| Add an integration recipe | `integration-recipes.md` |
| Fix a model name normalization | `error-recovery.md` |
| Update pricing | `pricing.md` |
| Add an app template | `app-scaffolding.md` |

### Style Guidelines

- **Agent-agnostic language** — say "the agent" not "Claude" or "Cursor"
- **Real API responses** — verify against live endpoints, don't guess field names
- **Plan-aware** — note which plan tier an endpoint requires
- **Cost-aware** — include per-call costs when documenting batch operations
- **Concise** — reference files should be scannable, not prose-heavy

### Testing Your Changes

Install the skill locally and test with your preferred agent:

```bash
npx skills add ./path-to-your-fork
```

Then try natural language queries that exercise the files you changed.

## Code of Conduct

This project follows the [Contributor Covenant Code of Conduct](CODE_OF_CONDUCT.md). By participating, you agree to uphold it.

## Questions?

- [Auto.dev API Docs](https://docs.auto.dev/)
- [Open an issue](https://github.com/drivly/auto-dev-skill/issues)
