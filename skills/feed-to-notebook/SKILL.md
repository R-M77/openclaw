---
name: feed-to-notebook
description: "Monitors RSS/Substack feeds and automatically creates NotebookLM notebooks with Audio Overviews for new articles. User-configurable sources, share targets, and schedules. Use when: (1) a user wants to track a Substack or RSS feed and get NotebookLM notebooks auto-generated for each new post, (2) running the scheduled feed check, (3) adding/removing feed sources, (4) manually triggering a check for new articles. Supports multiple concurrent sources with independent state tracking."
---

# feed-to-notebook

Monitors configured feeds, creates a NotebookLM notebook per new article, triggers Audio Overview generation, and shares with configured recipients.

## Config

Store source config at `memory/feed-to-notebook-config.json`. See `references/config-schema.md` for the full schema and examples.

Minimal example:

```json
{
  "sources": [
    {
      "id": "awg",
      "name": "The Innermost Loop",
      "substack_slug": "theinnermostloop",
      "share_with": ["you@gmail.com"],
      "notebooklm_account": "your-notebooklm-account@gmail.com",
      "state_file": "memory/feed-to-notebook-awg-state.json"
    }
  ]
}
```

## Workflow

### 1. Load Config

Read `memory/feed-to-notebook-config.json`. If missing, prompt user to create it (see `references/config-schema.md`).

### 2. Check Each Source for New Articles

For each source, use the appropriate fetcher:

**Substack** (preferred — catches paywalled posts):

```
GET https://{substack_slug}.substack.com/api/v1/posts?limit=5&sort=new
```

Returns JSON array. Compare `slug` of first item against state file.

**RSS fallback** (free posts only):

```
GET https://{substack_slug}.substack.com/feed
```

**Custom RSS**:

```
GET {rss_url}
```

If the latest article slug/URL matches state → nothing new, stop.

### 3. Fetch Article Text

Use `web_fetch` on the article URL. Substack public posts return full text without auth.

### 4. Create NotebookLM Notebook

Use `browser(profile=openclaw)`:

1. Navigate to `https://notebooklm.google.com/`
2. Click **Create new notebook**
3. Set title to the article title (use the `evaluate` action to set the input value and press Enter)
4. Click **Add sources** → **Copied text**
5. Paste the article text and click **Insert**
6. Click **Audio Overview** to trigger generation (do not wait)
7. Get notebook ID from the URL: `https://notebooklm.google.com/notebook/{NOTEBOOK_ID}`

### 5. Share Notebook

For each address in `share_with`:

1. Click **Share**
2. Type the email → Enter
3. Open the role dropdown → select **Editor**
4. Click **Send**

### 6. Update State

Write to the source's `state_file`:

```json
{
  "title": "<article title>",
  "url": "<article url>",
  "pubDate": "<ISO date>",
  "processedAt": "<ISO now>",
  "notebookId": "<id>",
  "notebookUrl": "https://notebooklm.google.com/notebook/<id>"
}
```

## Rules

- **One notebook per article** — never reuse an existing notebook
- Always use **Editor** access when sharing, never Viewer
- Use the Substack API over RSS when a `substack_slug` is configured
- If article text is behind a hard paywall (fetch returns login page), log a warning in state and skip

## Scheduling

To add a cron job for a source:

```bash
openclaw cron add --cron "30 9 * * *" --name "feed-to-notebook: {source.name}" \
  --session main --message "Run feed-to-notebook skill for source id: {source.id}"
```

Or run all sources at once:

```bash
openclaw cron add --cron "30 9 * * *" --name "feed-to-notebook: all" \
  --session main --message "Run feed-to-notebook skill: check all sources for new articles"
```

## Manual Triggers

- "Check AWG for new articles" → run for source id `awg`
- "Run feed-to-notebook" → run all sources
- "Add [Substack name] to feed-to-notebook" → add new source to config
