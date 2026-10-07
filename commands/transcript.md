---
description: Transcript of a YouTube video, then a short summary
argument-hint: YouTube URL or video id
allowed-tools:
  - Bash(hasdata *)
---

# /hasdata:transcript

Video: **$ARGUMENTS**

Take the video id from a watch URL (the value of `v`) or use the id the user pasted. Do not guess an id.

If a HasData tool is connected whose name contains `youtube_transcript`, call it and pass that id as `v`. Read the tool schema. Do not run a shell command when the tool exists.

If no HasData tool is connected, ask the user to press Connect. Do not install software and do not ask for an API key.

Use `hasdata youtube-transcript-api --help` only when the connector cannot be connected and the binary is already on PATH.

Summarize from the transcript text. Quote a line only when it appears in the transcript.
