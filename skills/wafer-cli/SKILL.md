---
name: wafer-cli
description: Operate the wafer CLI (@wafersecurity/cli) to manage Wafer projects, guardrails, limits, logs, analytics and guardrail tests. Use for any Wafer management task from the terminal. Prefer the CLI over raw HTTP — it handles auth and formatting, and every command supports --json for agents.
---

# wafer CLI

Agent-native CLI for the Wafer AI security gateway.

## Setup

```bash
npm i -g @wafersecurity/cli
wafer login                        # paste an API key (console → API keys); saved to ~/.wafer/config.json
# or: export WAFER_API_KEY=wafer_sk_...
wafer whoami                       # verify
```

`WAFER_URL` overrides the base (default `https://wafersecurity.ai`).

## Commands

```bash
wafer projects                          # list
wafer projects create "My App" --id my-app
wafer projects delete my-app

wafer guardrails get my-app
wafer guardrails set my-app pii redact  # rule: secrets|pii|blocklist|injection
wafer profile my-app set interrupt '{"guardrails":{"injection":{"action":"block"}}}'  # per-request posture (x-wafer-profile)
wafer cache my-app on
wafer ratelimit my-app 60               # or: off
wafer budget my-app 1000000             # or: off
wafer spend my-app 5                     # $5/day  (or: off, or: monthly 100)
wafer retry my-app 2 gpt-4o-mini         # retries + fallback model (or: off)
wafer webhook my-app https://ingest.heystack.dev/wafer   # stream decisions (or: off)
wafer logging my-app metadata          # content or metadata-only telemetry
wafer export my-app csv                   # export logs for audit (csv|json)

wafer test my-app "email me at jane@acme.com"   # run guardrails, no model call
wafer test my-app "..." --mode deterministic --profile enrich   # reproducible eval (no model calls)
wafer logs my-app --limit 20
wafer logs my-app --decision block --category secret --since 24h   # filtered history
wafer logs my-app --follow          # live tail (Ctrl+C to stop)
wafer analytics my-app              # usage + per-profile breakdown
wafer init my-app                       # print gateway base URLs for the SDK

wafer keys list | create <name> | revoke <id>
```

## For agents

- Add `--json` to any command for structured output. Parse that, not the human text.
- Non-zero exit code means failure; the error is on stderr (or `{"error": ...}` with `--json`).
- Typical flow to onboard an app: `wafer projects create` → `wafer init` (wire the base URL, see the `wafer-integrate` skill) → `wafer guardrails set` → `wafer test` to verify.
