---
name: hasdata-news
description: |
  Google News headlines and stories as structured JSON. Use this skill when the user wants recent news, headlines about a company, person, or topic, or a Google News search. Triggers on "news about", "latest headlines", "Google News for", "what is the news on". Prefer this over a general web search when the user asked for news.
allowed-tools:
  - Bash(hasdata *)
---

# Google News

## How to fetch

1. If a HasData tool is connected whose name contains `google_serp_news`, call it. Read its schema. Pass the topic in `q`. Do not run a shell command when the tool exists.
2. If no HasData tool is connected, ask the user to press Connect on the HasData connector for this chat. Do not install software and do not ask them to paste an API key.
3. Use the `hasdata` command only when the connector cannot be connected and the binary is already on PATH:

```bash
hasdata google-news --q "TOPIC" --pretty -o .hasdata/news.json
```

`gl` and `hl` set country and language. `topicToken`, `sectionToken`, `publicationToken`, and `storyToken` come from an earlier news response; do not invent them.

Return headline, source, date, and link from the tool output. Do not fill gaps with a web search.
