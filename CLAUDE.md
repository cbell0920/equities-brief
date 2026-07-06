# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

This is a **publish target**, not an application. It holds the static HTML output of the "Equities Morning Brief" — one of three daily TPTraders briefs (the others are Forex and Market Breadth, published to sibling repos `Forex-Brief` and `market-breadth`). There is no build step, no package.json, no tests — just hand/AI-authored static HTML pages pushed to `main` and served via GitHub Pages / raw GitHub links from the TPTraders hub.

The brief itself is generated elsewhere: a local Node server (`server.js`, in the parent `Claude Projects/` directory, not in this repo) fetches live prices from Yahoo Finance (`v8/finance/chart` per-symbol proxy — the old `v7/finance/quote` endpoint is dead/401s), then an AI pass adds analysis (regime call, per-ticker conviction scores, macro narrative, news, sector/options/earnings commentary) that isn't available from a free price feed. The finished HTML is committed to this repo as `index.html`.

## Repo structure

- `index.html` — the current/latest brief. **Overwritten each publish** — do not assume history lives here; check `archive/` or git log instead.
- `archive/` — dated snapshots of past briefs. Filenames are inconsistent (`index_625_eq.html`, `index_eq_0629.html`, `equities-brief-jun26.html`, etc. — mixed date formats, no fixed convention).
- `archive.html` — a hand-maintained index page linking into `archive/`. It is **stale**: as of the last check it only links one archived date even though `archive/` holds many more. Don't treat it as authoritative for what's archived — list the `archive/` directory instead.
- `equities-brief-dashboard.html` — the dashboard/template file the brief is built from (matches the styling of `index.html`: dark theme, tabbed layout — Overview / Watchlist / Macro Context / News / Analyst Note).
- `assets/` — images referenced by the dashboard (`quadrants.png`, `phases.jpg`).

## Brief HTML structure (`index.html` / `equities-brief-dashboard.html`)

Single self-contained HTML file, inline `<style>` and a small inline `<script>` for tab switching — no external JS dependencies. Sections, as tabs:

- **Overview** — market regime call, SPY/QQQ/VIX anchor table (support/resistance/bias/conviction score), top movers from the watchlist.
- **Watchlist** — a static 16-ticker table (TSLA, MSFT, NFLX, RBLX, V, META, SPCX, HOOD, BAC, GOOG, INTC, NVDA, ONDS, WMT, AAPL, VZ) with price, bias, levels, catalyst, conviction score.
- **Macro Context** — narrative writeup plus a key-levels grid.
- **News** — per-ticker/macro news items tagged bull/bear/neutral.
- **Analyst Note** — longer-form written commentary.

Each ticker row has a "→ Journal" link that deep-links into `tptraders.vercel.app` with a URL-encoded `brief_context` JSON blob (ticker, bias, key levels, stop, target, catalyst, VIX, regime) that pre-fills a trade journal entry in the separate TPTraders journal app.

Rows/prices marked `verify` mean the source data couldn't confirm a live price within 24 hours — this is a signal from the generation pipeline, not a placeholder to silently fill in.

## Working in this repo

- There's nothing to build, lint, or test — changes are just edits to static HTML.
- If asked to "publish a new brief," the correct flow is: fetch fresh data (via the external `server.js` pipeline) → generate updated HTML matching the existing tab structure/styling → overwrite `index.html` → commit → optionally save a dated copy under `archive/`.
- Match the existing inline-CSS style exactly (class names like `.regime-card`, `.conv-bar`, `.badge-bull`) rather than introducing new patterns, since `index.html` and `equities-brief-dashboard.html` are expected to stay visually identical.
