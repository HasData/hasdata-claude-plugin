---
name: hasdata-social
description: |
  Public Instagram profiles and recent posts: bio, follower and following counts, post counts, profile picture, and the posts on a handle. Use this skill when the user wants a public Instagram account, an audit of a profile, or follower stats. Triggers on "Instagram profile for", "follower count for", "Instagram bio for", "Instagram posts by", or a public Instagram handle. Returns structured JSON. YouTube is hasdata-youtube, TikTok is hasdata-tiktok, and Facebook is hasdata-facebook.
allowed-tools:
  - Bash(hasdata *)
---

# hasdata Instagram

Public Instagram profile and posts.

## When to use

- User asks about a public Instagram handle: bio, follower / following count, post count, profile pic
- They want recent posts from that handle

YouTube is [hasdata-youtube](../hasdata-youtube/SKILL.md). TikTok is [hasdata-tiktok](../hasdata-tiktok/SKILL.md). Facebook is [hasdata-facebook](../hasdata-facebook/SKILL.md). X and LinkedIn have no dedicated tool here; use [hasdata-scrape](../hasdata-scrape/SKILL.md) on a public profile URL only for those two.

## How to fetch

1. If a HasData tool is connected, call the one whose name contains `instagram_profile` or `instagram_posts`. Read its schema. Pass `handle` without an @ sign. Do not run a shell command when the tool exists. Do not call a tool whose name contains `instagram_comments`; that tool is not part of the connector this plugin ships.
2. If no HasData tool is connected, ask the user to press Connect on the HasData connector for this chat. Do not install software and do not ask them to paste an API key.
3. Use the `hasdata` command only when the connector cannot be connected and the binary is already on PATH.

## CLI fallback

```bash
hasdata instagram-profile --handle "hasdatadotcom" --pretty -o .hasdata/ig-hasdata.json
```

The handle must not include an @ sign. Only public profiles are supported.

## See also

- [hasdata-youtube](../hasdata-youtube/SKILL.md)
- [hasdata-tiktok](../hasdata-tiktok/SKILL.md)
- [hasdata-facebook](../hasdata-facebook/SKILL.md)
- [hasdata-scrape](../hasdata-scrape/SKILL.md) — X and LinkedIn public pages only
