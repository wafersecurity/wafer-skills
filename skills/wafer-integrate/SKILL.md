---
name: wafer-integrate
description: Wire an app's LLM calls through a Wafer gateway by changing the SDK base URL. Use when adding Wafer to a codebase that calls OpenAI or Anthropic, or when the user asks to route LLM traffic through Wafer / wafersecurity.ai.
---

# Integrate Wafer into a codebase

Goal: route the app's existing LLM calls through the project's Wafer gateway URL
**without changing the API key or any other code**.

## Steps

1. **Find the project's gateway URL.** Ask the user for the project id (or run
   `wafer init <project> --json`). The URLs are:
   - OpenAI: `https://wafersecurity.ai/p/<project>/openai`
   - Anthropic: `https://wafersecurity.ai/p/<project>/anthropic`

2. **Locate where the SDK client is constructed** (search for `OpenAI(`,
   `new OpenAI`, `Anthropic(`, `base_url`, `baseURL`, `OPENAI_BASE_URL`).

3. **Set the base URL only.** Keep the provider key as-is.

   Python (OpenAI):
   ```python
   client = OpenAI(api_key=OPENAI_KEY, base_url="https://wafersecurity.ai/p/<project>/openai")
   ```
   TypeScript (OpenAI):
   ```ts
   const client = new OpenAI({ apiKey: OPENAI_KEY, baseURL: "https://wafersecurity.ai/p/<project>/openai" });
   ```
   Anthropic:
   ```ts
   const client = new Anthropic({ apiKey: ANTHROPIC_KEY, baseURL: "https://wafersecurity.ai/p/<project>/anthropic" });
   ```
   Gemini (native google-genai SDK):
   ```python
   client = genai.Client(api_key=GEMINI_KEY, http_options={"base_url": "https://wafersecurity.ai/p/<project>/gemini"})
   ```

   OpenAI, Anthropic, Mistral and Gemini use their native SDKs/endpoints. Other
   providers (Groq, DeepSeek, xAI, Together, OpenRouter, Perplexity, Fireworks)
   are OpenAI-compatible — use the OpenAI SDK pointed at `/p/<project>/<provider>`.

   Prefer an env var so it's configurable: `OPENAI_BASE_URL` / a `WAFER_BASE_URL`.

4. **Verify** with a guardrail test (no model call):
   ```bash
   wafer test <project> "my email is jane@acme.com"   # expect a pii redaction
   ```

## Decision reports (optional)

For benchmarks/audits, send `x-wafer-report: summary` (decision headers on the
response: `x-wafer-decision`, `x-wafer-findings`, `x-wafer-guard-ms`) or
`x-wafer-report: detailed` (every guardrail runs synchronously; adds
`x-wafer-timings` per-check latency attribution; streaming responses carry the
full report as a trailing SSE event). Off by default — normal requests pay zero
report overhead.

## Rules

- Do **not** change or move the provider API key — Wafer passes it through.
- Do not hardcode secrets; read them from the existing env vars.
- Only edit the client construction; leave call sites untouched.
- If the app uses streaming, no change is needed — Wafer is streaming-aware.
- A blocked request returns HTTP `403` with `x-wafer-blocked: 1` and
  `x-wafer-categories` (the provider-shaped JSON body is caught by the SDK's own
  error handling). On a stream the block is a final `data:` event with the same
  `blocked` flag + `categories`. Map both to one "blocked" response.
