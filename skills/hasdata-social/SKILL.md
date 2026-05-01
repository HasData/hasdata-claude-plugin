---
name: hasdata-social
description: |
  Public Instagram profile data — bio, follower / following counts, post counts, profile picture, recent posts. Use this skill when the user wants to look up a public Instagram account, audit a profile, fetch follower stats, or pull the bio for a creator / brand. Triggers on "Instagram profile for", "follower count for", "look up @<handle>", "Instagram bio for", "scrape instagram for", or any public Instagram research. Returns structured JSON for a single profile.
allowed-tools:
  - Bash(hasdata *)
---

# hasdata Instagram API

Public Instagram profile data.

## When to use

- User asks about a single public Instagram handle
- They want bio, follower count, following count, post count, profile pic
- They're researching creators, brands, or competitors

For **non-Instagram social platforms**, no dedicated API exists — fall back to [hasdata-scrape](../hasdata-scrape/SKILL.md) on the public profile URL.

## API in this group

| Command              | Purpose                                  | Cost |
| -------------------- | ---------------------------------------- | ---- |
| `instagram-profile`  | Public profile data for a single handle  | 5    |

## Quick start

```bash
# Profile by handle (no @ symbol)
hasdata instagram-profile --handle "hasdatadotcom" --pretty -o .hasdata/ig-hasdata.json

# Multiple profiles in parallel
for h in nasa natgeo nike; do
  hasdata instagram-profile --handle "$h" --pretty -o ".hasdata/ig-$h.json" &
done
wait
```

## Tips

- **Handle is required** and must not include the `@` symbol.
- Only **public profiles** are supported. Private accounts return limited data.
- For **bulk audits** (e.g., compare 50 creators), fan out with `&` + `wait` — each call is 5 credits.

## Working with results

```bash
# Key stats
jq '{username, fullName, biography, followers: .followersCount, following: .followingCount, posts: .postsCount}' .hasdata/ig.json

# Compare follower counts across multiple
for f in .hasdata/ig-*.json; do
  jq -r '"\(.username)\t\(.followersCount)"' "$f"
done | sort -k2 -n -r
```

## See also

- [hasdata-scrape](../hasdata-scrape/SKILL.md) — for any other social platform's public profile pages
- [hasdata-search](../hasdata-search/SKILL.md) — `google-short-videos` for cross-platform short-form video research
