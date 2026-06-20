# Lead Discovery Engine
### A local-first prospect research tool, built as a Chrome extension

**Stack:** Vanilla JS · Manifest V3 · `chrome.storage.local` &nbsp;|&nbsp; **Size:** <1MB &nbsp;|&nbsp; **Status:** Active development (v1.5) &nbsp;|&nbsp; **Cost to run:** $0 — no APIs, no AI calls

---

![Lead Discovery Engine dashboard showing scored leads](screenshots/dashboard.png)

---

## The problem

Prospect research tools (Apollo, Clay, Apify-based scrapers, etc.) are subscription
products — often $50–300/month — for something that, at small scale, is really just:
search the web, pull structured data from public profiles, score it against criteria
you define, and manage the results. I wanted to see if I could build that pipeline
myself, end to end, as a single Chrome extension under 1MB, with zero ongoing cost
and zero external API dependency — no AI calls, no third-party data service, nothing
leaving the browser.

## What it does

The extension runs a full discovery-to-CRM pipeline inside Chrome:

- **Search-driven discovery** — takes a keyword, runs it through search results, and
  extracts public profile URLs using a multi-pass parser (it has to handle redirect-
  wrapped links, JSON-LD blocks, and several differently-shaped result layouts).
- **Profile enrichment** — visits each discovered profile and extracts structured data
  (bio, follower counts, links, etc.) using a cascading extraction strategy: structured
  data first, falling back through three other methods if the page doesn't have it.
- **Relevance scoring** — a weighted 0–100 scoring engine across five dimensions
  (keyword match, bio relevance, business signals, audience size fit, link quality),
  so results are ranked instead of just listed.
- **Built-in CRM dashboard** — a full management UI: status pipeline (New → Reviewed
  → Qualified → Contacted → Rejected), filtering by score/audience/business type,
  notes and follow-up dates per record, and CSV/Markdown export for use in any
  external tool.
- **CSV round-trip** — results can be exported and re-imported with status updates,
  so the tool can sit alongside a spreadsheet workflow instead of replacing it.

![Lead detail panel with five-dimension score breakdown](screenshots/lead-detail.png)

## Why it was a genuinely hard build

A few specific problems that took real debugging, not just glue code:

- **Storage quota safety.** `chrome.storage.local` has a hard 10MB ceiling. Long
  sessions saving hundreds of enriched records can approach it. I added a shared,
  quota-aware write path so a failed write logs a clear cause instead of throwing an
  opaque error mid-session, and added a merge strategy so a partial/degraded re-read
  of a profile can never silently overwrite good data already saved for that record.
- **Stale classification data.** An early link-type classifier mislabeled certain
  link types, and that bad label got baked into every record saved before the fix.
  Rather than requiring a full re-scrape, I built a recomputation layer that re-derives
  the correct type and link ranking from the raw URL at export time — so a CSV export
  is always correct, even for records saved months earlier.
- **Multi-pass extraction.** Search result pages render differently depending on
  layout experiments and redirect wrapping. The extractor runs five independent
  passes (anchor hrefs, citation text, data-attributes, structured data, full-page
  regex sweep) and de-duplicates the result, so it stays resilient as page markup
  shifts over time.

## Stack

Vanilla JavaScript, Manifest V3 (service worker + content scripts), `chrome.storage.local`
for persistence. No frameworks, no build step, no external APIs or AI calls — everything
runs locally in the browser.

## Status

Actively in development (currently v1.5). Source isn't public — happy to walk through
the code directly if you're interested.
