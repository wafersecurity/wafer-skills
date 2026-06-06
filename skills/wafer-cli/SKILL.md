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
wafer cache my-app on
wafer ratelimit my-app 60               # or: off
wafer budget my-app 1000000             # or: off

wafer test my-app "email me at jane@acme.com"   # run guardrails, no model call
wafer logs my-app --limit 20
wafer analytics my-app
wafer init my-app                       # print gateway base URLs for the SDK

wafer keys list | create <name> | revoke <id>
```

## For agents

- Add `--json` to any command for structured output. Parse that, not the human text.
- Non-zero exit code means failure; the error is on stderr (or `{"error": ...}` with `--json`).
- Typical flow to onboard an app: `wafer projects create` → `wafer init` (wire the base URL, see the `wafer-integrate` skill) → `wafer guardrails set` → `wafer test` to verify.
