# Apify Scraper Builder - a ChatGPT/Codex skill

A skill (ChatGPT/Codex format) for building, modifying, debugging, or reviewing **scraper Actors on Apify** - from configuring an existing no-code scraper to coding a full Crawlee-based crawler, wiring Pay-per-Event billing, and passing a publish checklist.

It is the **code-first entry point** for scraper Actors: given that you have already decided *what* to scrape, this skill helps you scaffold the Actor the right way and avoid the expensive mistakes (wrong template, charging on retries, `Actor.fail()` for business errors, burning your own IP during local dev).

## Why this skill exists

Most people walk into an Apify scraper thinking they should write everything from scratch with Crawlee. In about 60% of cases, a no-code scraper Actor or an LLM-extraction Actor is the right answer. Picking the wrong path - or the wrong Crawlee template - wastes compute, burns money on browser overhead you don't need, and produces an Actor that fails silently or charges users on errors.

This skill encodes the **three-paths decision** plus the five non-negotiable correctness rules every scraper Actor must follow, with empirically grounded defaults (memory tiers, rate-limit ratios, escalation triggers).

## What it can do

**Three creation paths:**
- **Path A - Configure a no-code scraper Actor:** one-off scrapes and prototypes via `apify/cheerio-scraper`, `apify/web-scraper`, `apify/puppeteer-scraper`, `apify/playwright-scraper`. No `apify create`, no Docker, no git.
- **Path B - AI extraction:** for content whose shape varies wildly between pages. Either configure `apify/ai-web-scraper` or roll your own Crawlee + LLM call. Includes the cost math that tells you when AI extraction breaks the unit economics.
- **Path C - Code from a Crawlee template:** the canonical production path. Full walkthrough from `apify create` to a deployed, monetized, observable scraper.

**The five things every scraper Actor must get right:**
1. Diagnose before coding (hunt for the internal JSON API first).
2. Always exit SUCCEEDED unless the platform crashed (graceful error records, never `Actor.fail()` for business errors).
3. PPE charge fires after success, never on retries.
4. Defensive selectors with layered fallbacks.
5. Cache external responses in the KV Store by default.

## What's inside

```
apify-scraper-builder/
├── SKILL.md                          # Orchestration hub: 3-paths decision + 5 correctness rules + common mistakes
├── README.md                         # This file
├── INSTALL.md                        # Installation instructions (ChatGPT/Codex + Claude Code)
├── agents/
│   └── openai.yaml                   # Codex agent interface + invocation policy
└── references/
    ├── path-a-nocode.md              # Configure apify/*-scraper; pageFunction templates; pagination; graduating to Path C
    ├── path-b-ai-extraction.md       # Pattern B1 (ai-web-scraper) vs B2 (Crawlee + LLM); prompts; cost math; reliability
    ├── path-c-crawlee-templates.md   # Full Path C: structure, schemas, routes, anti-bot escalation, KV state, graceful abort
    └── checklist-publish.md          # Pre-publish code review, local test, deploy, post-deploy + 7/30-day Insights checklist
```

## How to invoke

In Codex, the skill auto-activates on scraper-Actor requests (and can be forced with `$apify-scraper-builder`). Example prompts:

- "Build an Apify scraper for this e-commerce category page."
- "Should this be a no-code scraper, an AI-extraction Actor, or a Crawlee template?"
- "Review my Crawlee routes for PPE-charging correctness."
- "Why is my Actor's success rate stuck at 60%?"
- "Scaffold a `ts-crawlee-playwright-camoufox` Actor with graceful error handling and Pay-per-Event."

If auto-activation misses, force it: "Use the `apify-scraper-builder` skill to…"

## What's NOT inside

- **Anti-bot doctrine and the full tool ladder** (Cheerio → Playwright → Camoufox → managed APIs, vendor signatures, proxy economics): a separate scraping-doctrine layer. This skill cross-references it.
- **PPE pricing strategy** (what price to set, audit frameworks): a separate monetization layer.
- **README / Store description / input-schema copywriting**: a separate Apify Actor content layer.
- **MCP server Actors** (Standby mode, transports): a separate MCP-server-builder layer.

This skill is the scraper-Actor **code & architecture** entry point; it delegates doctrine, pricing, and content to their respective layers.

## Part of the mr-bridge.com toolkit

This skill is part of the [mr-bridge.com](https://mr-bridge.com) toolkit for scraping, data, and content automation. Related resources:

- [mr-bridge.com](https://mr-bridge.com) - home
- [Scrapers](https://mr-bridge.com/scrapers) - the Apify Actor portfolio
- [MCP servers](https://mr-bridge.com/mcp-servers) - Model Context Protocol servers
- [AI workflows](https://mr-bridge.com/ai-workflows) - agents and automation
- [Studies](https://mr-bridge.com/studies) - data studies and one-pagers
- [Articles](https://mr-bridge.com/articles) - write-ups and guides
- [Solutions](https://mr-bridge.com/solutions) - end-to-end solutions

## License

Personal use. Customize freely. No warranty.
