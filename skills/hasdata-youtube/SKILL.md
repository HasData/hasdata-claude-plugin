---
name: hasdata-youtube
description: |
  YouTube search, video details, channel pages, and full transcripts. Use this skill when the user wants to search YouTube, look up a video, see a channel's videos, shorts, playlists, or streams, or get the transcript or subtitles of a video. Triggers on "search YouTube for", "YouTube channel", "views on this video", "transcript of this video", "subtitles for", "summarize this YouTube video". Returns structured JSON. Fetch the transcript when the user wants what the video says.
allowed-tools:
  - Bash(hasdata *)
---

# YouTube

## How to fetch

1. If a HasData tool is connected, call the one whose name matches the task below. Read that tool's schema and fill the fields it asks for. Do not run a shell command when the tool exists.
2. If no HasData tool is connected, ask the user to press Connect on the HasData connector for this chat. Do not install software and do not ask them to paste an API key.
3. Use the `hasdata` command only when the connector cannot be connected and the binary is already on PATH.

## Which tool

| Ask | Tool name contains | Pass |
| --- | --- | --- |
| Search YouTube | `youtube_search` | `q` |
| One video's stats | `youtube_video` | `v` (the video id) |
| A channel page | `youtube_channel` | `channelId` |
| Transcript or subtitles | `youtube_transcript` | `v` (the video id) |

If the user gave a channel handle or a video URL and the tool wants an id, search first and take the id from the result. Do not guess an id.

To summarize a video, call the transcript tool and summarize that text. Do not ask the user to watch it.

## CLI fallback

```bash
hasdata youtube-search-api --help
hasdata youtube-video-api --help
hasdata youtube-channel-api --help
hasdata youtube-transcript-api --help
```
