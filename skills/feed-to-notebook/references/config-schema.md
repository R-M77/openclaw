# Config Schema

File location: `memory/feed-to-notebook-config.json`

## Full Schema

```json
{
  "sources": [
    {
      "id": "string (required, unique)",
      "name": "string (required, human-readable label)",

      "substack_slug": "string (optional) — e.g. 'theinnermostloop' for theinnermostloop.substack.com",
      "rss_url": "string (optional) — direct RSS URL for non-Substack feeds",

      "share_with": ["email@example.com"],
      "notebooklm_account": "string (optional) — Google account used for NotebookLM, defaults to logged-in account",

      "state_file": "string (required) — path to per-source state JSON, e.g. memory/feed-to-notebook-awg-state.json"
    }
  ]
}
```

**Source type:** provide either `substack_slug` OR `rss_url`, not both.

## Examples

### Single Substack source

```json
{
  "sources": [
    {
      "id": "awg",
      "name": "The Innermost Loop (AWG)",
      "substack_slug": "theinnermostloop",
      "share_with": ["you@gmail.com"],
      "notebooklm_account": "your-notebooklm-account@gmail.com",
      "state_file": "memory/feed-to-notebook-awg-state.json"
    }
  ]
}
```

### Multiple sources

```json
{
  "sources": [
    {
      "id": "awg",
      "name": "The Innermost Loop (AWG)",
      "substack_slug": "theinnermostloop",
      "share_with": ["me@gmail.com"],
      "state_file": "memory/feed-to-notebook-awg-state.json"
    },
    {
      "id": "stratechery",
      "name": "Stratechery",
      "substack_slug": "stratechery",
      "share_with": ["me@gmail.com", "partner@gmail.com"],
      "state_file": "memory/feed-to-notebook-stratechery-state.json"
    },
    {
      "id": "hacker-news-best",
      "name": "Hacker News Best",
      "rss_url": "https://hnrss.org/best",
      "share_with": ["me@gmail.com"],
      "state_file": "memory/feed-to-notebook-hn-state.json"
    }
  ]
}
```

## State File Schema (per source)

Written after each successful article processing:

```json
{
  "title": "Article Title",
  "url": "https://...",
  "pubDate": "2026-03-01T13:20:37Z",
  "processedAt": "2026-03-01T14:05:00Z",
  "notebookId": "903b2bc3-ee48-47b1-b950-1175caf66b9a",
  "notebookUrl": "https://notebooklm.google.com/notebook/903b2bc3-ee48-47b1-b950-1175caf66b9a"
}
```
