---
description: What Google AI Mode answers for a query, with the citations it shows
argument-hint: query
allowed-tools:
  - Bash(hasdata *)
---

# /hasdata:ai

Query: **$ARGUMENTS**

This is Google AI Mode, not the AI Overview box on a normal results page.

If a HasData tool is connected whose name contains `google_serp_ai_mode`, call it with `q`. Pass `gl`, `hl`, and `location` when the user named them. Read the tool schema. Do not run a shell command when the tool exists.

If no HasData tool is connected, ask the user to press Connect. Do not install software and do not ask for an API key.

Use `hasdata google-ai-mode --q "$ARGUMENTS"` only when the connector cannot be connected and the binary is already on PATH.

Return the answer and the cited links. Do not add sources the tool did not cite.

If the user asked for the AI Overview box instead, fetch the full results page first and expand it with the tool whose name contains `google_serp_ai_overview`, using the page token from that response.
