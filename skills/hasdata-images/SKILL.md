---
name: hasdata-images
description: |
  Google Images search results as structured JSON. Use this skill when the user wants image search results, picture URLs, or images of something from Google. Triggers on "image search", "Google Images for", "pictures of", "find images of". Returns image results from the images tool, not a screenshot of the results page.
allowed-tools:
  - Bash(hasdata *)
---

# Google Images

## How to fetch

1. If a HasData tool is connected whose name contains `google_images`, call it. Read its schema. Pass the query in `q`. Do not run a shell command when the tool exists.
2. If no HasData tool is connected, ask the user to press Connect on the HasData connector for this chat. Do not install software and do not ask them to paste an API key.
3. Use the `hasdata` command only when the connector cannot be connected and the binary is already on PATH:

```bash
hasdata google-images --q "QUERY" --pretty -o .hasdata/images.json
```

Show title, source page, and image URL from the results. `gl` and `hl` set country and language. `ijn` is the page index from a previous response.
