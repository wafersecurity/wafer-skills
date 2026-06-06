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
   - OpenAI: `https://wafersecurity.ai/p/<project>/openai/v1`
   - Anthropic: `https://wafersecurity.ai/p/<project>/anthropic`

2. **Locate where the SDK client is constructed** (search for `OpenAI(`,
   `new OpenAI`, `Anthropic(`, `base_url`, `baseURL`, `OPENAI_BASE_URL`).

3. **Set the base URL only.** Keep the provider key as-is.

   Python (OpenAI):
   ```python
   client = OpenAI(api_key=OPENAI_KEY, base_url="https://wafersecurity.ai/p/<project>/openai/v1")
   ```
   TypeScript (OpenAI):
   ```ts
   const client = new OpenAI({ apiKey: OPENAI_KEY, baseURL: "https://wafersecurity.ai/p/<project>/openai/v1" });
   ```
   Anthropic:
   ```ts
   const client = new Anthropic({ apiKey: ANTHROPIC_KEY, baseURL: "https://wafersecurity.ai/p/<project>/anthropic" });
   ```

   Prefer an env var so it's configurable: `OPENAI_BASE_URL` / a `WAFER_BASE_URL`.

4. **Verify** with a guardrail test (no model call):
   ```bash
   wafer test <project> "my email is jane@acme.com"   # expect a pii redaction
   ```

## Rules

- Do **not** change or move the provider API key — Wafer passes it through.
- Do not hardcode secrets; read them from the existing env vars.
- Only edit the client construction; leave call sites untouched.
- If the app uses streaming, no change is needed — Wafer is streaming-aware.
