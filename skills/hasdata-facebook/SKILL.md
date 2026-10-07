---
name: hasdata-facebook
description: |
  Public Facebook profile data. Use this skill when the user wants a public Facebook page or profile looked up by handle. Triggers on "Facebook page", "Facebook profile for", "look up this Facebook account". Returns structured JSON from the Facebook profile tool. There is no separate Facebook posts or comments tool; use the profile tool, and scrape the page URL only when the profile tool cannot answer the question.
allowed-tools:
  - Bash(hasdata *)
---

# Facebook

## How to fetch

1. If a HasData tool is connected whose name contains `facebook_profile`, call it. Read its schema. The handle is required. Do not run a shell command when the tool exists.
2. If no HasData tool is connected, ask the user to press Connect on the HasData connector for this chat. Do not install software and do not ask them to paste an API key.
3. Use the `hasdata` command only when the connector cannot be connected and the binary is already on PATH. Run `hasdata --help` and use a Facebook command only if it is listed.

Pass `handle` without a URL and without an @ sign. If the user pasted a facebook.com URL, take the page slug from the path.

This tool returns the public profile. It does not return a post feed or comments. Say so if the user asked for posts, and offer the profile fields the tool actually returned. Scrape the page with the web scraping tool only when the profile tool cannot answer.
