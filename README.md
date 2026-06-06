# Wafer skills

Agent skills for [Wafer](https://wafersecurity.ai) — the AI security gateway.
Install them into your coding agent (Claude Code, Cursor, …) with the
[`skills`](https://github.com/vercel-labs/skills) CLI:

```bash
npx skills add wafersecurity/wafer-skills --all
# or pick one:
npx skills add wafersecurity/wafer-skills --skill wafer-integrate
```

| Skill | Purpose |
|-------|---------|
| `wafer` | Overview + router for working with Wafer |
| `wafer-integrate` | Wire the gateway into a codebase (change the SDK base URL) |
| `wafer-guardrails` | Configure guardrails, cache, limits, budgets |
| `wafer-cli` | Drive the `@wafersecurity/cli` |
