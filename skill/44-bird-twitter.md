# Bird - X/Twitter CLI

**Location:** ` \openclaw\skills\bird\`  
**Emoji:** 🐦  
**Requirements:** `bird` CLI  
**Homepage:** https://bird.fast

## Purpose
Fast X/Twitter CLI using GraphQL + cookie-based authentication.

## Installation

**npm/pnpm/bun:**
```bash
npm install -g @steipete/bird
```

**Homebrew (macOS):**
```bash
brew install steipete/tap/bird
```

**One-shot (no install):**
```bash
bunx @steipete/bird whoami
```

## Authentication

Uses **cookie-based auth** (not API keys).

**Pass cookies directly:**
```bash
bird whoami --auth-token <token> --ct0 <ct0>
```

**Browser cookies:**
```bash
bird whoami --cookie-source chrome
```

**Check auth:**
```bash
bird check
```

**Arc/Brave:**
```bash
bird whoami --chrome-profile-dir <path>
```

## Account & Auth

```bash
bird whoami                    # Show logged-in account
bird check                     # Show credential sources
bird query-ids --fresh         # Refresh GraphQL query ID cache
```

## Reading Tweets

```bash
bird read <url-or-id>          # Single tweet
bird <url-or-id>               # Shorthand
bird thread <url-or-id>        # Full conversation
bird replies <url-or-id>       # List replies
```

## Timelines

```bash
bird home                      # Home (For You)
bird home --following          # Following timeline
bird user-tweets @handle -n 20 # User's profile
bird mentions                  # Your mentions
bird mentions --user @handle   # Someone else's mentions
```

## Search

```bash
bird search "query" -n 10
bird search "from:steipete" --all --max-pages 3
```

## News & Trending

```bash
bird news -n 10                # AI-curated from Explore
bird news --ai-only            # AI-curated only
bird news --sports             # Sports tab
bird news --with-tweets        # Include related tweets
bird trending                  # Alias for news
```

## Lists

```bash
bird lists                     # Your lists
bird lists --member-of         # Lists you're in
bird list-timeline <id> -n 20  # List tweets
```

## Bookmarks & Likes

```bash
bird bookmarks -n 10
bird bookmarks --folder-id <id>           # Specific folder
bird bookmarks --include-parent           # Include parent
bird bookmarks --author-chain             # Author's replies
bird bookmarks --full-chain-only          # Full thread
bird unbookmark <url-or-id>
bird likes -n 10
```

## Social Graph

```bash
bird following -n 20           # Who you follow
bird followers -n 20           # Your followers
bird following --user <id>     # Someone's following
bird about @handle             # Account origin/location
```

## Engagement

```bash
bird follow @handle
bird unfollow @handle
```

## Posting

```bash
bird tweet "hello world"
bird reply <url-or-id> "nice thread!"
bird tweet "check this out" --media image.png --alt "description"
```

**⚠️ Posting Warning:** More likely to be rate limited. If blocked, use browser tool instead.

## Media Uploads

```bash
# Single image
bird tweet "hi" --media img.png --alt "description"

# Multiple images (up to 4)
bird tweet "pics" --media a.jpg --media b.jpg

# Video (1 max)
bird tweet "video" --media clip.mp4
```

## Pagination

**Supported commands:** `replies`, `thread`, `search`, `bookmarks`, `likes`, `list-timeline`, `following`, `followers`, `user-tweets`

```bash
bird bookmarks --all                    # Fetch all pages
bird bookmarks --max-pages 3            # Limit pages
bird bookmarks --cursor <cursor>        # Resume from cursor
bird replies <id> --all --delay 1000    # Delay between pages (ms)
```

## Output Options

```bash
--json          # JSON output
--json-full     # JSON with raw API response
--plain         # No emoji, no color
--no-emoji      # Disable emoji
--no-color      # Disable ANSI colors (or NO_COLOR=1)
--quote-depth n # Max quoted tweet depth (default: 1)
```

## Global Options

```bash
--auth-token <token>           # auth_token cookie
--ct0 <token>                  # ct0 cookie
--cookie-source <source>       # Browser cookie source (repeatable)
--chrome-profile <name>        # Chrome profile name
--chrome-profile-dir <path>    # Profile dir or cookie DB path
--firefox-profile <name>       # Firefox profile
--timeout <ms>                 # Request timeout
--cookie-timeout <ms>          # Cookie extraction timeout
```

## Config File

**Location:** `~/.config/bird/config.json5` (global) or `./.birdrc.json5` (project)

```json5
{
  cookieSource: ["chrome"],
  chromeProfileDir: "/path/to/Arc/Profile",
  timeoutMs: 20000,
  quoteDepth: 1
}
```

**Environment variables:**
- `BIRD_TIMEOUT_MS`
- `BIRD_COOKIE_TIMEOUT_MS`
- `BIRD_QUOTE_DEPTH`

## Common Use Cases

**Check mentions:**
```bash
bird mentions -n 20 --json
```

**Search your tweets:**
```bash
bird search "from:@yourusername" --all
```

**Save bookmarks:**
```bash
bird bookmarks --all --json > bookmarks.json
```

**Monitor timeline:**
```bash
bird home --following -n 50
```

**Backup following:**
```bash
bird following --all --json > following.json
```

## Troubleshooting

### Query IDs Stale (404 errors)
```bash
bird query-ids --fresh
```

### Cookie Extraction Fails
- Check browser is logged into X
- Try different `--cookie-source`
- For Arc/Brave: use `--chrome-profile-dir`

### Rate Limited
- Wait before retrying
- Use browser tool for posting
- Avoid rapid-fire requests

## Best Practices

1. **Read/search freely** - less likely to hit limits
2. **Post carefully** - higher rate limit risk
3. **Use browser for posting** if rate limited
4. **Cache query IDs** - run `--fresh` when needed
5. **JSON output** for scripting/archiving

## Source
SKILL.md: ` \openclaw\skills\bird\SKILL.md`
