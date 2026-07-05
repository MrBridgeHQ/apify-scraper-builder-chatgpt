# Publish Checklist - Scraper Actor

Pre-publish + post-deploy verification for a scraper Actor on the Apify Store. Run this checklist before every `apify push` to production and again after deployment.

Complements (does not replace):
- Your README + Store description checklist (Apify Actor content reference layer)
- Your full revenue/billing audit framework (monetization reference layer)
- Your graceful-exit checklist (error-handling reference layer)
- Your shared cross-Actor publish checklist (applicable to all Actor types)

This file focuses on **scraper-specific** items.

## Pre-publish - code review

### Path selection

- [ ] You chose Path A (no-code), B (AI extraction), or C (Crawlee template) deliberately, not by default
- [ ] If Path C: you picked the right template (cheerio / playwright / camoufox) per the tool ladder, not the most powerful one
- [ ] Diagnostic done: you confirmed no internal JSON API exists and you know the target's anti-bot vendor

### `.actor/actor.json`

- [ ] `actorSpecification: 1`
- [ ] `name`, `title`, `description`, `version`, `buildTag` set
- [ ] `input: "./input_schema.json"` references existing file
- [ ] `dockerfile: "./Dockerfile"` references existing file
- [ ] `storages.dataset: "./dataset_schema.json"` if you have one
- [ ] `minMemoryMbytes` / `maxMemoryMbytes` tight to actual need (256–512 for Cheerio, 2048–4096 for Playwright)
- [ ] `defaultRunOptions.memoryMbytes` matches the typical run profile
- [ ] `defaultRunOptions.timeoutSecs` set (3600 for short scrapes, 7200+ for large crawls - never 0 for batch-mode scrapers)
- [ ] `categories` set (matches Apify Store taxonomy)
- [ ] No `usesStandbyMode` (that's for MCP servers, not scrapers - separate Actor type)

### `.actor/input_schema.json`

- [ ] Every property has `title`, `description`, `type`
- [ ] Required fields are listed under `required`
- [ ] `prefill` provided for every required field (this is what users see first)
- [ ] `editor` chosen correctly (`requestListSources` for URL arrays, `proxy` for proxy config, etc.)
- [ ] Min/max bounds on integer/array fields to prevent abuse
- [ ] Advanced fields wrapped in `sectionCaption` for collapsible UI groups
- [ ] No PII or default tokens in `prefill`

### `.actor/dataset_schema.json`

- [ ] Reflects the shape your scraper actually outputs
- [ ] All non-optional fields have `nullable: false`
- [ ] Optional fields are explicitly nullable
- [ ] Validates against your selector-strategy schema-validation doctrine

### `.actor/pay_per_event.json`

- [ ] One entry per chargeable event you call from code
- [ ] Each event has `eventTitle`, `eventDescription`, `eventPriceUsd`
- [ ] No declaration of `apify-actor-start` (platform-managed)
- [ ] If you declare `apify-default-dataset-item`, you do NOT also declare a custom event for the same dataset items (would double-charge)
- [ ] Price points cross-checked against your pricing-strategy doctrine for the market

### Dockerfile

- [ ] Base image is the lightest that includes what you need (Cheerio: `apify/actor-node:20`; Playwright: `apify/actor-node-playwright-chrome:20`; Camoufox: `apify/actor-node-playwright-firefox:20` + camoufox install)
- [ ] `npm install --omit=dev --omit=optional` (skip dev deps in production image)
- [ ] No `playwright install` if the base image already has it
- [ ] `.dockerignore` excludes `tests/`, `storage/`, `node_modules`, `.git`

### `src/main.ts`

- [ ] `await Actor.init()` at the top
- [ ] Input validated; bad input → graceful exit (push error to dataset, SUCCEEDED, NOT `Actor.fail()`)
- [ ] State persisted via `Actor.on('migrating')` / `Actor.on('aborting')` if the run is long
- [ ] Proxy configuration created with the right group (`DATACENTER` by default)
- [ ] Crawler config: `useSessionPool: true`, `persistCookiesPerSession: true`, `retryOnBlocked: true`, `maxRequestsPerCrawl` set, `maxRequestsPerMinute` set
- [ ] Outer try/catch that catches all errors and pushes an error record to the dataset, then `await Actor.exit()` (SUCCEEDED)
- [ ] `Actor.setStatusMessage(..., { isStatusMessageTerminal: true })` on every exit path
- [ ] **NO** `Actor.fail()` for business-logic errors - only for true infrastructure failures
- [ ] **NO** `process.exit()` anywhere

### `src/routes.ts`

- [ ] Every label has a handler (`router.addHandler('LABEL', ...)`)
- [ ] Default handler exists (`router.addDefaultHandler(...)`) and logs unrouted requests
- [ ] Extraction is delegated to pure functions in `src/extractors/*.ts` (testable)
- [ ] Junk results discarded before charging (`if (!product.name || !product.url) continue;`)
- [ ] Charging via atomic `Actor.pushData(data, 'event-name')` where applicable
- [ ] Every charge result is checked for `eventChargeLimitReached` and aborts via `crawler.autoscaledPool?.abort()` when reached
- [ ] No `Actor.charge()` calls in catch blocks or on error paths
- [ ] Empty pages logged + pushed as error records, not silently dropped

### Selectors

- [ ] Multi-selector fallbacks for every critical field (`'h2, [itemprop="name"], .product-name'`)
- [ ] Prefer schema.org (`[itemprop="..."]`) over visual class names where possible
- [ ] URL resolution via `new URL(href, baseUrl).href` - never string concat
- [ ] Relative paths handled (`/foo/bar` and `https://...` both work)
- [ ] Empty / missing values return `null`, not `undefined` or empty string

### Tests

- [ ] Fixtures captured (HTML scrubbed of PII) in `tests/fixtures/`
- [ ] Vitest config exists and `npm test` runs without network access
- [ ] At least one test per critical extractor function
- [ ] At least one "missing fields" test (verify nulls propagate correctly)
- [ ] `Actor.charge()`, `Actor.pushData()`, `Actor.openKeyValueStore()` mocked in tests
- [ ] For local development: **NO local network calls to the target site** - anywhere

### Secrets & hygiene

- [ ] No API tokens / secret keys in `actor.json`, `package.json`, source files
- [ ] Secret env vars set in Apify Console UI (Actor settings → Environment variables → secret)
- [ ] `.gitignore` includes `.env`, `secrets/`, `credentials.json`, `storage/`
- [ ] `.dockerignore` includes the same plus `tests/`, `.git`, `node_modules`

### Source-code confidentiality

`apify push` uploads everything not listed in `.gitignore`, and the public Actor API exposes the uploaded source file names to authenticated users (the `isSourceCodeHidden` flag only hides them in the Console UI, not in the API). So keep internal material out of the pushed Actor.

- [ ] Internal notes, history files, audit/strategy docs are in `.gitignore` (or stored outside the Actor directory)
- [ ] After `apify push`, run: `apify actors info <username>/<slug> --json | jq '.versions[0].sourceFiles[].name'`
- [ ] Source file list contains ONLY public files (README, CHANGELOG, package.json, package-lock.json, src/main.js, src/routes.js, .actor/*, Dockerfile, .gitignore, .dockerignore)
- [ ] Set `isSourceCodeHidden: true` so source files are hidden from the public UI
- [ ] Verify in incognito browser that the Source code tab is gone

## Pre-publish - local test

```bash
# Run tests (no network)
npx vitest run

# Local Apify run with small input
cat > storage/key_value_stores/default/INPUT.json <<EOF
{ "startUrls": [{ "url": "https://example-shop.com/category/test" }], "maxProducts": 5 }
EOF
apify run --purge
```

- [ ] All tests pass
- [ ] Local run completes without errors
- [ ] `storage/datasets/default/` contains the expected items
- [ ] Log shows `Charged event ...` lines on successful items (no-op locally but logged)
- [ ] Log shows NO charges on error paths
- [ ] Bad input (empty `startUrls`) → SUCCEEDED with error record, NOT FAILED

## Deploy

```bash
apify login
apify push
```

- [ ] Build completes without errors
- [ ] Build log mentions the correct base image
- [ ] Image size is reasonable (Cheerio < 500 MB, Playwright < 1.5 GB)
- [ ] No warnings about missing files in the build

## Post-deploy - verification

### Apify Console

- [ ] Build appears as the latest version
- [ ] **Monetization** tab shows your PPE events with correct prices
- [ ] **Source code** tab is hidden (you've set `isSourceCodeHidden: true`)
- [ ] Input form renders correctly (every field has the right editor, prefills work)

### Smoke test in the Console

Start a small test run from the Console:

- [ ] Status: SUCCEEDED (not FAILED, not TIMED-OUT)
- [ ] Dataset has the expected number of items
- [ ] Each item has all required fields populated (>95%)
- [ ] **Billing** tab shows N events charged (matches dataset size)
- [ ] Per-event price matches `pay_per_event.json`
- [ ] `apify-actor-start` event fires once
- [ ] Memory plateau well under the configured cap
- [ ] Run time is within the expected range

### Bad-input smoke test

Start a run with intentionally bad input (empty `startUrls`, invalid URL, etc.):

- [ ] Status: SUCCEEDED (NOT FAILED - this is the iron rule)
- [ ] Dataset contains an error record (`error: true`, `errorType: 'INVALID_INPUT'`, `message: ...`)
- [ ] Status message ends with terminal text ("Error: ... See dataset.")
- [ ] No PPE events charged

### Selector-failure smoke test

Run against a URL where the target's HTML has changed (or against an unrelated page):

- [ ] Status: SUCCEEDED
- [ ] Dataset contains an `EXTRACTION_FAILED` record
- [ ] No PPE events charged on empty pages
- [ ] Status message indicates the issue

### Anti-bot smoke test (if applicable)

For Camoufox/Playwright scrapers, run on a known-protected page:

- [ ] Status: SUCCEEDED
- [ ] Either dataset has items OR contains a `TARGET_BLOCKED` error
- [ ] If blocked: no charges fired
- [ ] If succeeded: charge count matches item count

## After publishing to the Store

- [ ] README has been audited per your Apify Actor content doctrine
- [ ] Pricing has been audited per your monetization audit framework
- [ ] Discounts (Bronze/Silver/Gold) opt-in is enabled in the Monetization wizard
- [ ] Created an "Examples" entry in the Actor's Store page (one realistic input + expected output sample)

## After 7 days in the Store

- [ ] **Insights → Run success rate** ≥ 95% (below = code bugs OR error-handling not graceful)
- [ ] **Insights → Average run time** within expectations
- [ ] **Insights → Compute units per run** stable (no leaks)
- [ ] No unaddressed user reports in **Issues** tab
- [ ] Reviews appearing in **Reviews** tab - respond to constructive feedback within 48h
- [ ] **Selector breakage canary**: pick a known-good target URL; if extraction rate drops below 95%, the site changed - open an issue + update selectors

## After 30 days

- [ ] **Pricing audit**: per your monetization audit framework, compare actual cost-per-item against PPE price. Adjust if margin is wrong.
- [ ] **Cost-per-item** by template:
  - Cheerio: < $0.0005/item (compute + proxy)
  - Playwright: < $0.005/item
  - Camoufox: < $0.02/item
  - AI extraction: highly variable, depends on token usage - see `references/path-b-ai-extraction.md` cost math
- [ ] If margin is < 50%: raise price (note: a price-increase delay applies before it takes effect)
- [ ] If runs failing: instrument with more `Actor.setStatusMessage` calls and structured error records

## Iron rules - do not skip

If any of these fail, **delete the deployment** and fix before re-publishing:

- [ ] No secrets visible in Apify Source code tab (incognito browser check)
- [ ] No internal notes / history docs in source files
- [ ] No `Actor.charge()` calls in error paths
- [ ] No `Actor.fail()` calls for business-logic errors
- [ ] `eventChargeLimitReached` handled in every chargeable code path
- [ ] No `process.exit()` or `Actor.exit()` mid-crawl - use `crawler.autoscaledPool?.abort()`
- [ ] Tests pass without network access
