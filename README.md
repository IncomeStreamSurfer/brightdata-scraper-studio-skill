# Bright Data Scraper Studio — Claude Code skill

A skill that teaches Claude Code, Cursor or Codex how to build, run and self-heal custom web
scrapers for any public site, entirely from the terminal.

Point your coding agent at a URL, describe the data you want in plain English, and get back a
working scraper you own — plus a Collector ID that is already a live API endpoint.

> **Try Bright Data:** [brdta.com/iss25](https://brdta.com/iss25) — get **$25 of credit** to
> run everything in this skill.
>
> *That's my affiliate link. It costs you nothing extra and helps support work like this.*

## Install

```bash
git clone https://github.com/IncomeStreamSurfer/brightdata-scraper-studio-skill.git ~/.claude/skills/brightdata-scraper-studio
```

That's it. Claude Code picks the skill up on the next session — no config, no restart.

**Cursor / Codex:** clone anywhere and point your rules file at `SKILL.md`, or copy its
contents into `.cursor/rules` or `AGENTS.md`.

**Verify it loaded:** ask your agent *"what do you know about Bright Data Scraper Studio?"* —
it should reference the five scraper types without searching the web.

## What it covers

- **Setup in three commands** — runs via `npx`, so no global install, no `sudo`, no `EACCES`,
  no node version manager fights
- **Picking the right scraper type** — PDP, Discovery, Discovery + PDP, Search, and Sitemap,
  with the rule for naming the shape explicitly so you don't pay the build time twice
- **The build loop** — `create` → `run` → `heal` → `approve`
- **Self-healing** — when a site changes its HTML, one prompt repairs the scraper you own and
  keeps the same Collector ID, so every downstream trigger and schedule keeps working
- **Pinning the Collector ID** in your rules file so agents stop rebuilding the scraper on
  every session and just run it
- **The trigger API** — batch and real-time paths, callable from any language or scheduler
- **A "check for a faster path first" step** — `scrape`, `search`, `discover` and 43 pre-built
  `pipelines` all return in seconds, so the agent doesn't burn 15 minutes rebuilding something
  that already exists

## Why the gotchas section matters

Every gotcha in the skill is a failure hit in real use, not a guess:

- zsh globs on unquoted URLs — `...?v=abc` dies with `no matches found` before the CLI runs
- piping a long-running command into `head` SIGPIPEs the poller and destroys the result
- stale docs examples return `{"error": "dead_page"}` rather than failing loudly
- `scraper create` takes 5–25 minutes, so an agent must never start one without telling you

## Requirements

- Node.js 20+
- A Bright Data account — [brdta.com/iss25](https://brdta.com/iss25) for $25 of credit

No global install needed. The skill runs everything through `npx -p @brightdata/cli`.

## Files

| File | Contents |
|------|----------|
| `SKILL.md` | The skill — setup, type selection, build loop, trigger API, gotchas |
| `reference.md` | Full command surface, flags, SERP JSON shape, all 43 pipeline types, credential paths |

## Cost

$1.50 per 1K page loads pay-as-you-go, down to $1.00 at scale. Sign up through
[brdta.com/iss25](https://brdta.com/iss25) for $25 of credit — that's roughly 16,000 page
loads, far more than you need to work through everything here. Run `bdata budget` to check
your balance.

## Contributing

Issues and PRs welcome — especially new gotchas found in real use.

## License

MIT
