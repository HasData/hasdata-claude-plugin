---
name: hasdata-shorts
description: |
  Google short-video results: YouTube Shorts, TikToks, and Reels as they appear in Google. Use this skill when the user wants short videos ranking for a query, or which clips Google shows for a keyword. Triggers on "short videos for", "YouTube Shorts ranking", "Google short videos", "Reels in Google results". A transcript of one YouTube video is hasdata-youtube. A TikTok profile is hasdata-tiktok.
allowed-tools:
  - Bash(hasdata *)
---

# Google short videos

## How to fetch

1. If a HasData tool is connected whose name contains `google_serp_short_videos`, call it. Read its schema. `q` is required. Do not run a shell command when the tool exists.
2. If no HasData tool is connected, ask the user to press Connect on the HasData connector. Do not install software and do not ask for an API key.
3. Use `hasdata google-short-videos` only when the connector cannot be connected and the binary is already on PATH.

Pass `gl`, `hl`, `location`, and `deviceType` when the user names them. `page` starts at 0. For a time window, `tbs` takes `qdr:h`, `qdr:d`, `qdr:w`, `qdr:m`, or `qdr:y`.

Return title, source, and link. Do not describe a video the tool did not list.

One YouTube video's transcript is [hasdata-youtube](../hasdata-youtube/SKILL.md).
