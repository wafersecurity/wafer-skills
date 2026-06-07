---
name: wafer
description: Overview and router for working with Wafer, the AI security gateway. Use when a project routes LLM traffic through Wafer (a wafersecurity.ai gateway URL or WAFER_API_KEY is present), or when adding security guardrails (PII redaction, secret detection, prompt-injection defense, caching, rate limits) to an LLM app.
---

# Wafer — AI security gateway

Wafer sits between an app and the LLM provider. You point your existing OpenAI or
Anthropic SDK at a project's **gateway URL** (keeping your own API key), and Wafer
applies guardrails to every request and response.

A project's gateway URLs are uniform — `https://wafersecurity.ai/p/<project>/<provider>`:

```
OpenAI:    https://wafersecurity.ai/p/<project>/openai
Anthropic: https://wafersecurity.ai/p/<project>/anthropic
Gemini:    https://wafersecurity.ai/p/<project>/gemini
Mistral:   https://wafersecurity.ai/p/<project>/mistral
```

Other providers: groq, deepseek, xai, together, openrouter, perplexity, fireworks.

## Which skill to use

- **Wiring Wafer into a codebase** (change the SDK base URL) → use `wafer-integrate`.
- **Configuring guardrails / limits / cache** for a project → use `wafer-guardrails`.
- **Driving Wafer from the terminal** (CLI commands, `--json`) → use `wafer-cli`.

## Core facts

- **Bring your own key (BYOK):** the provider key passes straight through; Wafer never stores it.
- **One-line integration:** only the base URL changes — no SDK to learn.
- **Guardrails:** secrets, PII, blocklist, prompt-injection, semantic cache, rate limit, daily token budget, USD spend limit (per day/month). Each guardrail is `block | redact | flag | off`.
- **Reliability:** optional provider retries (429/5xx) with backoff and an optional same-provider fallback model.
- **Observability:** stream decisions to a webhook sink (native heystack.dev integration) and export request logs as CSV/JSON for audit. Webhook events are metadata-only.
- **Programmatic access:** the admin API and CLI authenticate with a `WAFER_API_KEY` (create one in the console → Agents → API keys).
- **Console:** https://console.wafersecurity.ai · **Site:** https://wafersecurity.ai
