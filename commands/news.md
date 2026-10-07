---
description: Recent Google News headlines for a topic
argument-hint: topic
allowed-tools:
  - Bash(hasdata *)
---

# /hasdata:news

Topic: **$ARGUMENTS**

If a HasData tool is connected whose name contains `google_serp_news`, call it and pass the topic in `q`. Read the tool schema. Do not run a shell command when the tool exists.

If no HasData tool is connected, ask the user to press Connect. Do not install software and do not ask for an API key.

Use `hasdata google-news --q "$ARGUMENTS"` only when the connector cannot be connected and the binary is already on PATH.

Return headline, source, date, and link. Do not add stories the tool did not return.
