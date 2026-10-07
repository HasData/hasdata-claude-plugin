---
name: hasdata-ai-overview
description: |
  The Google AI Overview box on a normal results page. Use this skill when the user wants the AI Overview for a keyword, whether Google shows an AI Overview, or the sources cited inside that box. Triggers on "AI Overview", "AI Overviews for", "does this keyword have an AI overview", "sources in the AI overview". Fetch the full Google results page first, then expand the overview with the page token from that response. The token expires in about a minute. This is not Google AI Mode. AI Mode is hasdata-ai-mode.
allowed-tools:
  - Bash(hasdata *)
---

# Google AI Overview

## How to fetch

Do this in one turn, in order. The token dies in about a minute.

1. If HasData tools are connected, call the full results tool whose name contains `google_serp_serp` and does not contain `light`. Pass `q`, plus `gl`, `hl`, `location`, or `deviceType` when the user named them.
2. Read the response for an `aiOverview` block and its `pageToken`. If the block is missing, say this query returned no AI Overview for that country, language, and device. Stop. Do not invent an overview and do not switch to AI Mode unless the user asked for AI Mode.
3. Immediately call the tool whose name contains `google_serp_ai_overview` with that `pageToken`. The schema requires `pageToken` and does not take a query.

If no HasData tool is connected, ask the user to press Connect. Do not install software and do not ask for an API key.

There is no separate CLI command for this second step. Do not call `google-ai-mode` and present it as the AI Overview.

Return the overview text and the cited links from the overview tool.
