# Bright Data CLI — reference

## Full command surface

| Command | Purpose |
|---------|---------|
| `login` / `logout` | OAuth (`-k` key, `-d` device flow for SSH), clear credentials |
| `scraper create/run/heal/approve` | Scraper Studio lifecycle |
| `scrape <url>` | Web Unlocker — one page as clean markdown |
| `search <query>` | SERP API — Google/Bing/Yandex as structured JSON |
| `discover <query>` | AI intent search: fans out to multiple queries, reranks |
| `pipelines <type> <url>` | Pre-built structured extractors |
| `browser` | Remote browser sessions |
| `budget` / `zones` | Balance, per-zone spend, zone management |
| `config get\|set` / `init` | CLI config, setup wizard |
| `skill list\|add` | Install Bright Data agent skills |
| `add mcp --agent <a>` | Wire the MCP server into a coding agent |
| `status <job-id>` | Check an async snapshot job |

Global flags: `-k, --api-key <key>`, `--timing`, `-v, --version`.
Output flags on most commands: `-o <path>`, `--json`, `--pretty`.

## Command flags that matter

`search` — `--engine google|bing|yandex`, `--country <code>`, `--language <code>`,
`--page <n>` (0-indexed), `--type web|news|images|shopping`, `--device desktop|mobile`.

`scraper heal` — `--url <url>` (verification target), `--auto-approve`,
`--timeout <seconds>` (default 600), `--max-retries <n>` (default 4), `--no-retry`.

`scraper approve` — `--reject`, `--url <url>`, `--timeout <seconds>`.

`pipelines` — `--format json|csv|ndjson|jsonl`, `--timeout <seconds>`.

## SERP JSON shape

`bdata search --json` returns far more than ten links. Top-level keys:

- `organic` — ranked results (`rank`, `title`, `link`, `description`)
- `people_also_ask` — PAA questions
- `related` — related searches
- `navigation`, `pagination`, `general` (includes `results_cnt`)
- Feature blocks appear conditionally: `snack_pack` + `snack_pack_map` (local pack),
  `hotels_selection`, and others depending on query intent

Not every key is present on every query — always use `.get()` rather than direct indexing.

Useful for SEO work: `site:domain.com <brand>` queries reveal directory listing gaps, but
**verify hits by scraping the page** — `site:` matches produce false positives on
similarly-named entities.

## Pre-built pipeline types

`bdata pipelines list` is authoritative. As of writing, 43 types:

```
amazon_product, amazon_product_reviews, amazon_product_search, apple_app_store,
bestbuy_products, booking_hotel_listings, crunchbase_company, ebay_product,
etsy_products, facebook_company_reviews, facebook_events,
facebook_marketplace_listings, facebook_posts, github_repository_file,
google_maps_reviews, google_play_store, google_shopping, homedepot_products,
instagram_comments, instagram_posts, instagram_profiles, instagram_reels,
linkedin_company_profile, linkedin_job_listings, linkedin_people_search,
linkedin_person_profile, linkedin_posts, reddit_posts, reuter_news,
tiktok_comments, tiktok_posts, tiktok_profiles, tiktok_shop, walmart_product,
walmart_seller, x_posts, yahoo_finance_business, youtube_comments,
youtube_profiles, youtube_videos, zara_products, zillow_properties_listing,
zoominfo_company_profile
```

These return deep records. `youtube_videos` on one URL returns 45 fields including the full
transcript as plain text and as `formatted_transcript` with per-line start/end/duration in ms,
plus engagement counts, tags, category, and recommended videos.

## Trigger API

Auth on every call: `Authorization: Bearer $BRIGHTDATA_API_KEY`, `Content-Type: application/json`.

### Batch — any number of URLs

```
POST https://api.brightdata.com/dca/trigger?collector=c_...&queue_next=1
```

Body is a JSON **array** matching the collector's input schema (default: a `url` field):

```json
[{"url": "https://example.com/a"}, {"url": "https://example.com/b"}]
```

Returns `{"collection_id": "j_...", "start_eta": "..."}` immediately.

Optional query params: `version=dev`, `name=<label>`, `queue=<group>`, `deadline=1h|30m|45s`,
`no_downloads=1`, `confirm_cancel=1`, `notify=<url-encoded JSON>`, `deliver=<url-encoded JSON>`.

Then poll every 5s:

```
GET https://api.brightdata.com/dca/dataset?id=j_...
```

Returns `{"status": "building"}` while running, then the JSON array of results.

### Real-time — single URL

```
POST https://api.brightdata.com/dca/crawl?collector=c_...&timeout=50s
```

`timeout` is **required**, between `25s` and `50s`. Body is a **single object**, not an array.
`200` returns the data directly. `202` means it exceeded the window and returns
`{"error": "crawl_results_timeout", "response_id": "..."}` — fall back to the batch path.

### ID conventions

- `c_*` — Collector ID, the published scraper. Stable across heals.
- `j_*` — Collection/snapshot ID, one run.
- `sd_*` — snapshot ID returned by `pipelines`.

## Credential storage

| Platform | Path |
|----------|------|
| macOS | `~/Library/Application Support/brightdata-cli/` |
| Linux | `~/.config/brightdata-cli/` |
| Windows | `%APPDATA%\brightdata-cli\` |

`credentials.json` (restricted perms) and `config.json` (zones, prefs).
Resolution order: CLI flags → env vars → `config.json` → defaults.

## Docs

- Coding-agent prompts: https://docs.brightdata.com/datasets/scraper-studio/coding-agent-prompts
- Build with the CLI: https://docs.brightdata.com/datasets/scraper-studio/build-with-the-cli
- CLI overview: https://docs.brightdata.com/cli/overview
- Scraper type walkthrough: https://docs.brightdata.com/datasets/scraper-studio/ai-agent
- API quickstart: https://docs.brightdata.com/api-reference/scraper-studio-api/Getting_started_with_the_API
- Free tier: https://docs.brightdata.com/general/account/billing-and-pricing/free-tier
