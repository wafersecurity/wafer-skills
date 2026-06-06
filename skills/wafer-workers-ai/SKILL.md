---
name: wafer-workers-ai
description: Instrument Cloudflare Workers AI (env.AI binding) calls with Wafer guardrails using @wafersecurity/workers-ai. Use when an app is a Cloudflare Worker that calls env.AI.run(...) and needs PII/secret redaction, blocklist or prompt-injection defense — the HTTP base_url approach does not apply to binding calls.
---

# Instrument Cloudflare Workers AI bindings

`env.AI.run(...)` is an in-process binding call, so the Wafer gateway's
`base_url` approach can't intercept it. Use the wrapper instead.

## Steps

1. Install: `npm i @wafersecurity/workers-ai`
2. Ensure a `WAFER_API_KEY` is available to the Worker (create one in the
   console → Agents → API keys; add it as a secret: `wrangler secret put WAFER_API_KEY`).
3. Instrument. Two ways:

   **One line (no per-call changes)** — wrap the default export; set
   `WAFER_PROJECT` + `WAFER_API_KEY` secrets. Every `env.AI.run` is auto-guarded:
   ```js
   import { withWafer } from "@wafersecurity/workers-ai";
   export default withWafer(handler);
   ```

   **Explicit** — wrap the binding and use `ai.run`:
   ```js
   import { wrapAI, WaferBlockedError } from "@wafersecurity/workers-ai";
   const ai = wrapAI(env.AI, { project: "<project>", apiKey: env.WAFER_API_KEY, ctx });
   const res = await ai.run("@cf/meta/llama-3.3-70b-instruct", { messages });
   ```

- `ai.run(model, inputs, options)` is a drop-in for `env.AI.run`.
- Pass the Worker's `ctx` (ExecutionContext) so logging is non-blocking.
- Blocked inputs throw `WaferBlockedError`; catch it and return a 4xx.

## How it behaves

- Tier-0 guardrails (PII, secrets, blocklist, injection heuristic) run locally —
  no added latency; redaction happens before the model sees the prompt.
- The optional LLM judge runs via the Worker's own `env.AI` binding (no extra
  Wafer round-trip) against the policy set in the console.
- Policy is fetched from the project config (cached); analytics are logged to
  Wafer (metadata only — no prompt/response content; set `log:"off"` to disable).
- Fail-open: if Wafer is unreachable, the AI call still runs.

## When NOT to use this

If the app calls models over HTTP (OpenAI/Anthropic/Gemini/etc. SDKs), use the
gateway base URL instead — see the `wafer-integrate` skill.
