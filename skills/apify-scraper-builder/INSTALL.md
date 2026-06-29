# Installation — Apify Scraper Builder skill (ChatGPT/Codex)

This skill ships in the **ChatGPT/Codex** skill format: a `SKILL.md`, an `agents/openai.yaml` interface descriptor, and a `references/` folder. It has no runtime dependencies — it is pure documentation/doctrine the agent reads, so there is nothing to `pip install`.

## Prerequisites

- ChatGPT Codex (or another agent that reads the Codex skill format), OR Claude Code
- A terminal with `unzip` (macOS, Linux) or PowerShell with `Expand-Archive` (Windows)

To actually build and deploy a scraper Actor you'll also want, on your dev machine:

- Node.js LTS (18+ recommended)
- The Apify CLI: `npm install -g apify-cli`
- An Apify account (`apify login`)

## Install for Codex

Codex discovers skills from its skills directory. Place the `apify-scraper-builder/` folder there.

### macOS / Linux

```bash
# 1. Create the Codex skills directory if it doesn't exist
mkdir -p ~/.codex/skills

# 2. Unzip the skill into that directory
unzip apify-scraper-builder.zip -d ~/.codex/skills/

# 3. Verify the structure
ls ~/.codex/skills/apify-scraper-builder/
# Expected: SKILL.md  README.md  INSTALL.md  agents/  references/
```

### Windows (PowerShell)

```powershell
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.codex\skills"
Expand-Archive -Path .\apify-scraper-builder.zip -DestinationPath "$env:USERPROFILE\.codex\skills\"
Get-ChildItem "$env:USERPROFILE\.codex\skills\apify-scraper-builder\"
```

The `agents/openai.yaml` file declares the display name, short description, default prompt, and `allow_implicit_invocation: true` (so the skill auto-activates on scraper-Actor requests).

## Install for Claude Code (alternative)

The same content works as a Claude Code skill. Claude Code ignores `agents/openai.yaml` and reads only `SKILL.md` + `references/`.

```bash
# macOS / Linux
mkdir -p ~/.claude/skills
unzip apify-scraper-builder.zip -d ~/.claude/skills/
ls ~/.claude/skills/apify-scraper-builder/
```

```powershell
# Windows (PowerShell)
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.claude\skills"
Expand-Archive -Path .\apify-scraper-builder.zip -DestinationPath "$env:USERPROFILE\.claude\skills\"
```

## Verification

Open your agent and ask:

> "What skills do you have access to?"

You should see `apify-scraper-builder` in the list. If not, confirm `SKILL.md` is at:

- Codex (macOS/Linux): `~/.codex/skills/apify-scraper-builder/SKILL.md`
- Claude Code (macOS/Linux): `~/.claude/skills/apify-scraper-builder/SKILL.md`
- Windows: the equivalent under `%USERPROFILE%`

## First test

In any agent session, ask:

> "I want to scrape an e-commerce category page on Apify. Should I use a no-code scraper, AI extraction, or a Crawlee template?"

The skill should activate, route to the three-paths decision, and recommend a path with a template choice and the memory/rate-limit defaults.

## Updating the skill

Replace the directory with the newer version:

```bash
# Codex (macOS / Linux)
rm -rf ~/.codex/skills/apify-scraper-builder
unzip apify-scraper-builder.zip -d ~/.codex/skills/
```

## Uninstalling

```bash
# Codex (macOS / Linux)
rm -rf ~/.codex/skills/apify-scraper-builder

# Claude Code (macOS / Linux)
rm -rf ~/.claude/skills/apify-scraper-builder
```

```powershell
# Windows (PowerShell)
Remove-Item -Recurse -Force "$env:USERPROFILE\.codex\skills\apify-scraper-builder"
```

## Troubleshooting

**Skill doesn't activate when expected.**
Auto-activation depends on the `description` in `SKILL.md`'s frontmatter and (for Codex) `allow_implicit_invocation` in `agents/openai.yaml`. Triggers include "build a scraper on Apify", "scrape X with an Actor", "apify create --template ts-crawlee-*", Crawlee/Cheerio/Playwright/Camoufox, request queues, session pools, pagination. If your phrasing doesn't match, force activation: "Use the `apify-scraper-builder` skill to…"

**The skill references doctrine layers it doesn't ship.**
By design. This skill is the scraper-Actor *code & architecture* entry point; it cross-references separate doctrine layers (anti-bot strategy, PPE pricing, Actor content, MCP servers) that live in their own skills. The cross-references are descriptive — the code patterns and checklists here are self-contained.

## Skill content overview

For the full skill layout, see [`README.md`#whats-inside](README.md#whats-inside).
