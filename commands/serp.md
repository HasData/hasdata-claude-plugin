---
description: Full Google results page for a keyword, including rankings, ads, and the local pack
argument-hint: keyword, plus country or city if it matters
allowed-tools:
  - Bash(hasdata *)
---

# /hasdata:serp

Keyword: **$ARGUMENTS**

If a HasData tool is connected whose name contains `google_serp_serp` and does not contain `light`, call it. Read its schema. `q` is the keyword. Pass `gl`, `hl`, `location`, and `deviceType` when the user named a country, language, city, or phone versus desktop. Do not run a shell command when the tool exists.

If no HasData tool is connected, ask the user to press Connect. Do not install software and do not ask for an API key.

Use `hasdata google-serp --q "$ARGUMENTS"` only when the connector cannot be connected and the binary is already on PATH.

Return organic position, title, link, and snippet. Also list ads, people also ask, and the local pack when the response includes them. If an aiOverview block is present, say so.
