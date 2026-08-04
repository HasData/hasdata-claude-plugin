---
name: hasdata-social
description: |
  Public Instagram and YouTube data. Instagram: bio, follower / following counts, post counts, profile picture, recent posts. YouTube: search results, video details, channel pages (videos, shorts, playlists, streams), and full video transcripts. Use this skill when the user wants to look up a public Instagram account, audit a profile, fetch follower stats, search YouTube, pull stats for a video or channel, or get the transcript / subtitles of a YouTube video. Triggers on "Instagram profile for", "follower count for", "look up @<handle>", "Instagram bio for", "search YouTube for", "YouTube channel stats", "views on this video", "transcript of this video", "subtitles for", "summarize this YouTube video", or any public social-platform research. Returns structured JSON.
allowed-tools:
  - Bash(hasdata *)
---

# hasdata social APIs

Public Instagram profile data plus YouTube search, video, channel, and transcript data.

## When to use

- User asks about a single public Instagram handle: bio, follower / following count, post count, profile pic
- They want to search YouTube, or pull stats and metadata for a specific video or channel
- They want the transcript of a YouTube video, for example to summarize or quote it
- They're researching creators, brands, or competitors on either platform

Transcripts are the cheapest way to get a video's content: fetch the transcript instead of
asking the user to watch, then summarize from the text.

For **other social platforms** (TikTok, X, LinkedIn, Facebook), no dedicated API exists.
Fall back to [hasdata-scrape](../hasdata-scrape/SKILL.md) on the public profile URL.

## APIs in this group

| Command                  | Purpose                                                        | Cost |
| ------------------------ | -------------------------------------------------------------- | ---- |
| `instagram-profile`      | Public profile data for a single handle                        | 5    |
| `youtube-search-api`     | YouTube search results with duration / date / feature filters  | 10   |
| `youtube-video-api`      | Full data for one video by video ID                            | 10   |
| `youtube-channel-api`    | Channel page by ID or handle, per tab (videos, shorts, ...)    | 10   |
| `youtube-transcript-api` | Full transcript / subtitles for one video                      | 10   |

## Quick start

```bash
# Instagram profile by handle (no @ symbol)
hasdata instagram-profile --handle "hasdatadotcom" --pretty -o .hasdata/ig-hasdata.json

# Search YouTube, newest first, long-form only
hasdata youtube-search-api --q "web scraping tutorial" --date month --sort-by date \
  --pretty -o .hasdata/yt-search.json

# One video's details, then its transcript
hasdata youtube-video-api --v-param "dQw4w9WgXcQ" --pretty -o .hasdata/yt-video.json
hasdata youtube-transcript-api --v-param "dQw4w9WgXcQ" --pretty -o .hasdata/yt-transcript.json

# Channel uploads by handle
hasdata youtube-channel-api --channel-id "@hasdatadotcom" --tab videos \
  --pretty -o .hasdata/yt-channel.json
```

## Tips

- **Instagram handle is required** and must not include the `@` symbol. Only public profiles
  are supported; private accounts return limited data.
- **YouTube video ID is the `v=` value**, 11 characters, not the whole watch URL. For
  `https://www.youtube.com/watch?v=dQw4w9WgXcQ` pass `dQw4w9WgXcQ`.
- **Channel ID accepts either form**: the canonical `UC…` ID or the public `@handle`.
- **Transcripts:** add `--type asr` for the auto-generated track, or `--language-code de` for
  a specific language. The requested track has to exist on the video, so retry without
  `--language-code` if it comes back empty.
- **Channel tabs** change the response shape: `featured` (default), `videos`, `shorts`,
  `playlists`, `streams`, `posts`, `podcasts`, `releases`.
- **Paginate** channel and search results with `--pagination-token` from the previous response.
- For **bulk work** (e.g., 50 creators), fan out with `&` + `wait`.

## Working with results

```bash
# Instagram key stats
jq '{username, fullName, biography, followers: .followersCount, following: .followingCount, posts: .postsCount}' .hasdata/ig.json

# Transcript as plain text, ready to summarize
jq -r '.transcript[].snippet' .hasdata/yt-transcript.json | tr '\n' ' '

# Transcript with timestamps
jq -r '.transcript[] | "\(.startTimeText) \(.snippet)"' .hasdata/yt-transcript.json

# Compare follower counts across multiple profiles
for f in .hasdata/ig-*.json; do
  jq -r '"\(.username)\t\(.followersCount)"' "$f"
done | sort -k2 -n -r
```

## See also

- [hasdata-scrape](../hasdata-scrape/SKILL.md) — for any other social platform's public pages
- [hasdata-search](../hasdata-search/SKILL.md) — `google-short-videos` for cross-platform
  short-form video research across YouTube, TikTok, and Instagram
