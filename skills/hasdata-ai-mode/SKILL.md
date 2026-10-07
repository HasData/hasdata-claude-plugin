---
name: hasdata-ai-mode
description: |
  Google AI Mode answers, with the citations Google shows in that mode. Use this skill when the user wants what Google AI Mode says, a follow-up in the same AI Mode thread, or an AI answer for a query in a given country and language. Triggers on "AI Mode", "Google AI Mode", "what does Google AI say", "ask Google AI". This is not the AI Overview box on a normal results page. That box is hasdata-ai-overview.
allowed-tools:
  - Bash(hasdata *)
---

# Google AI Mode

## How to fetch

1. If a HasData tool is connected whose name contains `google_serp_ai_mode`, call it. Read its schema. `q` is required. Do not run a shell command when the tool exists.
2. If no HasData tool is connected, ask the user to press Connect on the HasData connector. Do not install software and do not ask for an API key.
3. Use `hasdata google-ai-mode` only when the connector cannot be connected and the binary is already on PATH.

Pass `gl`, `hl`, and `location` when the user names a country, language, or city. Do not invent `uule`.

A follow-up in the same AI Mode thread uses `continuable` and `subsequentRequestToken` from the previous AI Mode response. Do not start a new query when the user is continuing that thread.

Return the answer and the cited links from the tool. Do not add sources the tool did not cite.

The AI Overview box on a normal Google page is a different tool. It needs a token from the full results page and is [hasdata-ai-overview](../hasdata-ai-overview/SKILL.md). Do not call AI Mode to fake an AI Overview.
