---
name: brightdata-scraper-studio
description: Use when setting up Bright Data Scraper Studio, building or running a custom web scraper for any site, fixing a scraper that broke after a site redesign, or using the Bright Data CLI (bdata/brightdata) for scraping, SERP results, or structured extraction. Triggers on "scraper studio", "bright data", "bdata", "build a scraper", "collector ID", "scraper broke", "scrape this site".
---

# Bright Data Scraper Studio

AI builds a custom scraper for any public site from a plain-English description. You own the
generated code, it runs on Bright Data's unblocking network, and when the target site changes
its HTML you heal it with one prompt instead of rewriting selectors.

Everything happens in the terminal. No dashboard required.

## Setup

Run the CLI through `npx` — no global install, no sudo, no EACCES, no node version fights.

```bash
alias bdata="npx -p @brightdata/cli bdata"
bdata login          # browser OAuth, one time
bdata budget         # confirms auth, shows balance
```

`bdata` and `brightdata` are the same binary. Requires Node 20+.

Credentials land in `~/Library/Application Support/brightdata-cli/credentials.json`
(macOS), `~/.config/brightdata-cli/` (Linux). Never print that file or the token.

Headless / CI — skip `login` entirely:

```bash
export BRIGHTDATA_API_KEY=<key>
```

First-ever login auto-provisions two zones (`cli_unlocker`, `cli_browser`). Existing
accounts won't see this.

## Check for a faster path first

**Do not build a scraper before checking whether one of these already answers the question.**
Building takes 5–25 minutes; these return in seconds.

| Need | Command |
|------|---------|
| One page as clean markdown | `bdata scrape <url>` |
| Google/Bing/Yandex SERP as JSON | `bdata search "<query>" --country ie --json` |
| AI-ranked results across many queries | `bdata discover "<question>"` |
| A major site already covered | `bdata pipelines <type> <url> --pretty` |

`bdata pipelines list` shows the pre-built extractors (Amazon, LinkedIn, TikTok, Instagram,
YouTube, Reddit, Zillow, Walmart, eBay, Etsy, Crunchbase, Google Maps, X, and more). These are
maintained for you. Scraper Studio is for **everything else** — regional e-commerce, B2B
catalogs, niche verticals, docs sites, internal tools, competitor changelogs.

See [reference.md](reference.md) for the full pipeline list and the trigger API.

## Pick the scraper type

This is the one decision that determines whether the scraper does what you want.

| Type | You give it | You get back |
|------|-------------|--------------|
| PDP | a list of item URLs | one row per URL, full detail |
| Discovery | a category or listing URL | every item on that listing |
| Discovery + PDP | a category URL | every item, plus full detail each |
| Search | a keyword (+ optional country) | Discovery or Discovery+PDP shape |
| Sitemap | a site URL | full per-page detail for every URL on the site |

Say the type explicitly in the description you pass to `create` — "crawl the entire sitemap
and return…", "discover every product on this listing and return…". Ambiguous descriptions
produce the wrong shape and you pay the build time twice.

Sitemap is the right choice for docs sites, blogs and changelogs feeding a RAG pipeline.

## Build, run, heal

```bash
# 1. Build. 5-15 min typical, up to 25 on complex targets. Returns a Collector ID (c_*).
bdata scraper create <url> "<what to extract, and which of the 5 shapes>"

# 2. Run.
bdata scraper run <collector_id> <url> --pretty
bdata scraper run <collector_id> <url> -o out/data.json

# 3. Site changed? Heal in place. Keeps the same Collector ID. Under 1,000 chars.
bdata scraper heal <collector_id> "<what broke>" --url <url>

# 4. Review the approval envelope, then commit.
bdata scraper approve <collector_id> --url <url>
bdata scraper approve <collector_id> --reject
```

Heal also **extends** a working scraper — ask it to capture extra fields alongside the
existing ones. It stops at an approval gate by default; `--auto-approve` for unattended runs.
`--max-retries <n>` (default 4) handles HTTP 429.

## Pin the Collector ID

After a successful build, record the ID in the project's rules file (`CLAUDE.md`,
`.cursor/rules`, `AGENTS.md`). This is what stops agents rebuilding the scraper every session.

```
SCRAPER_STUDIO_COLLECTOR_ID=c_your_id_here
USAGE="bdata scraper run $SCRAPER_STUDIO_COLLECTOR_ID <url> --pretty"
TARGET=https://...
```

Healing preserves the ID, so every downstream trigger and schedule keeps working.

## The Collector ID is a live API endpoint

There is no deployment step. The scraper you just built is already callable from any
codebase, language or scheduler:

```bash
curl -X POST "https://api.brightdata.com/dca/trigger?collector=$COLLECTOR_ID&queue_next=1" \
  -H "Authorization: Bearer $BRIGHTDATA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '[{"url": "https://example.com/a"}, {"url": "https://example.com/b"}]'
```

Returns a `j_*` collection id; poll `GET /dca/dataset?id=j_...` every 5s until it stops
returning `{"status":"building"}`. For a single URL under 50s, use `/dca/crawl` instead —
it returns the data directly. Details in [reference.md](reference.md).

## Gotchas

These are verified failures, not hypotheticals.

- **Quote every URL.** zsh globs on `?` and `&` — an unquoted `...?v=abc` dies with
  `no matches found` before the CLI ever runs.
- **Never pipe a long-running command into `head`.** It SIGPIPEs the poller mid-run and you
  lose the result. Use `-o <file>` and read the file.
- **Verify example URLs before demoing.** Docs examples go stale; a dead target returns
  `{"error": "dead_page"}` rather than failing loudly at the CLI level.
- **`create` is slow by design.** Budget 5–25 minutes and say so up front. Never kick one off
  without telling the user it will take that long and cost credits.
- **Public data only.** No login-walled or paywalled targets.
- **Mask keys.** Don't cat `credentials.json` or the full `.env` on screen.

## Cost

$1.50 per 1K page loads pay-as-you-go, down to $1.00 at scale. Flat CPM. Every account gets
5,000 free credits per month, automatically applied. `bdata budget` shows balance and pending
charges.

## Related

`bdata skill list` / `bdata skill add <name>` installs Bright Data's own agent skills
(search, scrape, data-feeds, scraper-builder, bright-data-mcp, best-practices) into Claude
Code, Cursor or Codex. `brightdata add mcp --agent claude-code` wires up the MCP server if you
want the full 60-tool surface instead of the CLI.
