---
description: Google results for a query, as structured data
argument-hint: query
allowed-tools:
  - Bash(hasdata *)
---

# /hasdata:search

Query: **$ARGUMENTS**

If a HasData tool is connected whose name contains `google_serp_serp_light`, call it with the query. If the user asked for ads, the knowledge panel, or related searches, call the tool whose name contains `google_serp_serp` and does not contain `light`. Read the tool schema. Do not run a shell command when the tool exists.

If no HasData tool is connected, ask the user to press Connect. Do not install software and do not ask for an API key.

Use `hasdata google-serp-light --q "$ARGUMENTS"` only when the connector cannot be connected and the binary is already on PATH.

Return title, link, and snippet from the tool output.
