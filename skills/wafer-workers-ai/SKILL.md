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
   `WAFER_PROJECT` + `WAFER_API_KEY` secrets. Every `env.AI.run` is auto-guarded
   across ALL handlers (fetch, queue, scheduled, tail) — safe for cron + queue
   consumers:
   ```js
   import { withWafer } from "@wafersecurity/workers-ai";
   export default withWafer(handler);   // handler can have fetch/queue/scheduled
   ```

   **Explicit** — wrap the binding and use `ai.run`:
   ```js
   import { wrapAI, WaferBlockedError } from "@wafersecurity/workers-ai";
   const ai = wrapAI(env.AI, { project: "<project>", apiKey: env.WAFER_API_KEY, ctx });
   const res = await ai.run("@cf/meta/llama-3.3-70b-instruct", { messages });
   ```

- `ai.run(model, inputs, options)` is a drop-in for `env.AI.run`.
- Pass the Worker's `ctx` (ExecutionContext) so logging is non-blocking.
- Blocked inputs throw `WaferBlockedError`; catch it and return a 4xx. The error
  exposes `.categories` (e.g. `["pii","injection"]`) — the same value the gateway
  sends as the `x-wafer-categories` header, so one handler covers binding + gateway.
- A mid-stream block (secrets/PII/blocklist set to block) terminates the stream
  with a final `data:` event carrying the legacy `response` marker AND the
  unified `{wafer:{blocked:true,categories:[…]}}` object — identical shape to
  the gateway's in-band stream block.

## How it behaves

- Tier-0 guardrails (PII, secrets, blocklist, injection heuristic) run locally —
  no added latency; redaction happens before the model sees the prompt.
- The optional LLM judge runs via the Worker's own `env.AI` binding (no extra
  Wafer round-trip) against the policy set in the console.
- Pass `profile: "<name>"` in opts to apply a named posture override from the
  project (the binding equivalent of the gateway's `x-wafer-profile` header),
  or pass `(model, inputs) => name` to select a profile per binding call.
- Pass `onDecision(report)` to consume decision, category, profile, finding and
  timing metadata in application code. Callback failures never affect inference.
- Policy + telemetry settings are fetched from the project config (cached).
  Logs model, decision, findings, tokens and latency; request/response content
  is captured when the project enables it (override: `log:"metadata"` never,
  `log:"content"` always, `log:"off"` disable).
- Speech-to-text (Whisper / Deepgram): the returned transcript is screened by the
  **input** guardrails before it feeds a downstream model — spoken injection or
  blocklisted phrases are caught. A blocked transcript throws `WaferBlockedError`;
  a redacted one rewrites `res.text`.
- Fail-open: if Wafer is unreachable, the AI call still runs.
- Streaming (`{ stream: true }`): chunks pass through with zero added latency; the
  full response is captured and logged after the stream ends. Block-action
  guardrails still cut the stream on a hit; mid-stream redaction isn't applied.

## When NOT to use this

If the app calls models over HTTP (OpenAI/Anthropic/Gemini/etc. SDKs), use the
gateway base URL instead — see the `wafer-integrate` skill.
