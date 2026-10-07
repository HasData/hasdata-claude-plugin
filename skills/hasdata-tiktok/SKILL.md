---
name: hasdata-tiktok
description: |
  Public TikTok profiles, posts, search, and comments. Use this skill when the user wants a TikTok account, that account's posts, a TikTok search, or comments on a TikTok video. Triggers on "TikTok profile", "TikTok for", "search TikTok", "TikTok comments", "what has this creator posted on TikTok". Returns structured JSON from the TikTok tools. Do not scrape the TikTok page instead.
allowed-tools:
  - Bash(hasdata *)
---

# TikTok

## How to fetch

1. If a HasData tool is connected, call the one whose name matches the task below. Read that tool's schema and fill the fields it asks for. Do not run a shell command when the tool exists.
2. If no HasData tool is connected, ask the user to press Connect on the HasData connector for this chat. Do not install software and do not ask them to paste an API key.
3. Use the `hasdata` command only when the connector cannot be connected and the binary is already on PATH. Run `hasdata --help` and use a TikTok command only if it is listed.

## Which tool

| Ask | Tool name contains | Pass |
| --- | --- | --- |
| Profile | `tiktok_profile` | `handle` (no @ sign) |
| Posts from an account | `tiktok_posts` | `handle` |
| Search videos or accounts | `tiktok_search` | `keyword` |
| Comments on a video | `tiktok_comments` | `videoId` |

Search uses `keyword`, not `q`. If the user gave a video URL and the comments tool wants `videoId`, take the id from that URL. Do not invent engagement numbers.
