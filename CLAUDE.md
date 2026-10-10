# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this project is

A static personal portfolio site for Christopher Hendry (software engineering, formerly games development). Plain HTML/CSS/JS, no build tooling, no framework, no package.json. Every page is a hand-written `.html` file that shares one stylesheet (`styles.css`) and one small script (`script.js`). Served directly by GitHub Pages from this repo — there is no compiled or generated output, so the file on disk is exactly what ships.

## Common commands

- **No build step.** Edit the HTML/CSS/JS files directly and reload the browser.
- **Local preview:** serve the folder rather than opening files with `file://`, since some tools (and relative asset/script loading) don't behave correctly under `file://`. From the repo root:
  ```
  python -m http.server
  ```
  then open `http://localhost:8000/index.html` (or whichever page).
- **Deploy:** GitHub Pages reads straight from this repo — there is no `.github/workflows` directory and no `CNAME` file, so this is a plain "Pages reads from a branch" setup (confirm in the repo's GitHub Settings → Pages which branch/folder it's pointed at — likely `main`, root). Pushing to that branch is the deploy; there is nothing else to run.

## High-level architecture

### Shared nav (duplicated markup — update every file)

Every one of the 7 top-level `.html` files repeats the identical nav block near the top of `<body>`:

```html
<nav class="site-nav">
  <div class="site-nav-inner">
    <a class="brand" href="index.html">Christopher Hendry</a>
    <a href="backtester.html">Backtester</a>
    <a href="sentrywatch.html">Sentrywatch</a>
    <a href="docs-assistant.html">docs-assistant</a>
    <a href="mobile-app.html">Mobile App</a>
    <a href="widget.html">Widget</a>
    <a href="games-development.html">Games Development</a>
  </div>
</nav>
```
The only difference per file is which link carries `class="active"`. **There is no shared include/template** — if you add, remove, or rename a project page, you must hand-edit this exact block in all 7 files (`index.html`, `backtester.html`, `sentrywatch.html`, `docs-assistant.html`, `mobile-app.html`, `widget.html`, `games-development.html`). It is easy to update 6 and forget one; grep for `site-nav-inner` to check you got all of them.

### Case-study page pattern

Each project page (everything except `index.html`) follows the same structure. Looked at `sentrywatch.html`, `backtester.html`, `docs-assistant.html`, `mobile-app.html`, `widget.html` to confirm — `games-development.html` follows it too but swaps screenshots for video embeds (see below).

- `<header class="page-header">` — eyebrow "Case Study", `<h1>` project name, one-paragraph `.tagline`.
- One or more `.case` blocks inside a `<section>`. Each `.case` is one sub-project/feature and has:
  - `.case-head` — `<h3>` title + a `<span class="kind">` (e.g. "Solo · open-core, shipped", "3-person team") describing scope/status.
  - Optional `<p class="summary">` — only present when the case has one cohesive paragraph of framing (used when there's a single `.case` on the page, e.g. `docs-assistant.html`, `mobile-app.html`, `widget.html`; the multi-case pages like `sentrywatch.html`/`backtester.html` mix cases with and without a summary).
  - `.stack` — a row of `<span>` pills naming the tech used (e.g. `<span>Python</span><span>FastAPI</span>`).
  - `.log` — a `<ul>` of `<li>` entries, each `<span class="tag">Found/Diagnosed/Designed/Built/Verified/Measured/Owned/Researched/Documented/Optimised/Fixed</span>` followed by a `<p>` describing what happened. This is the real substance of each case study — a changelog-style account of a specific bug found, a decision made, or something verified live.
  - `.gallery` (or `.gallery-wide` / `.gallery-single`) — a grid of `<figure><img loading="lazy"><figcaption></figure>` blocks. Three gallery widths exist in `styles.css`: plain `.gallery` (phone-shaped 9:19.5 crop, for mobile screenshots), `.gallery-wide` (16:10, for desktop/dashboard screenshots), `.gallery-single` (one fixed-width column, for a single terminal screenshot).
- `games-development.html` has no screenshots at all — it uses `.video-embed` (a YouTube `<iframe>`) + `.video-caption` instead, because that work predates the screenshot convention and is better shown as gameplay video.
- Pages with a gallery include `<div class="lightbox" id="lightbox">...</div>` plus `<script src="script.js"></script>` just before `</body>` (click an image to zoom, click/Escape to close). `index.html` and `games-development.html` omit both, since neither has a `.gallery`.

### Screenshots (`screenshots/`)

All 26 images live flat in `screenshots/` (no subfolders), named `<project>-<thing>.png`/`.jpg` (e.g. `sentrywatch-dashboard-overview.png`, `docs-assistant-eval.jpg`). I found no `CLAUDE_NOTES.txt` or similar convention file in this repo — the convention below is inferred from the images' own `alt` text, which is explicit about it: several captions/alt strings say things like *"Terminal output of the real eval suite run against both Ollama and the Claude API..."* and *"Terminal transcript of a real MCP client connecting to the Sentrywatch MCP server..."*. So: the terminal-style screenshots (`.jpg` files mostly) are **real captured program output, rendered as styled HTML and then screenshotted in a browser** — not fabricated/mocked-up terminal graphics. The `.png` files are mostly real app/dashboard UI screenshots. If you add a new screenshot, keep following this real-output-only rule; don't invent a fake terminal render.

### `styles.css` theming

- Dark theme is the default, defined as CSS custom properties on `:root` (`--bg`, `--ink`, `--accent`, etc.).
- A light variant is defined twice: once under `@media (prefers-color-scheme: light)` guarded by `:root:not([data-theme="dark"])` (so OS-level light mode applies automatically), and again under an explicit `:root[data-theme="light"]` selector.
- **No toggle control exists anywhere in the HTML or `script.js`.** Nothing in this repo ever sets `data-theme` on `<html>`/`<body>` — grepped for `data-theme`, `theme-toggle`, `localStorage` and found no setter, only the two CSS selectors above. So today the site only follows the OS's `prefers-color-scheme`; the `data-theme="light"`/`"dark"` hooks are dormant plumbing for a manual override switch that was never wired up. Don't assume a toggle exists.
- Fonts: Inter (body/headings) and IBM Plex Mono (nav, tags, labels, monospace-feeling UI chrome), loaded from Google Fonts via `@import` at the top of `styles.css`.

## File/directory map

- `index.html` — home page: hero/intro, the 6-card project grid linking to every case study, education, and core-skills grid.
- `backtester.html` — case study: the live multi-broker trading system ("Chopper"), its LangGraph daily briefing, the desktop installer, and the copy-trading signal engine.
- `sentrywatch.html` — case study: the Windows watchdog + dashboard, its Flutter mobile companion app, the AI incident copilot, and the MCP server.
- `docs-assistant.html` — case study: the hybrid-retrieval RAG assistant over the other projects' own docs.
- `mobile-app.html` — case study: the Flutter companion app that reuses the trading system's backend via API.
- `widget.html` — case study: the Android (Jetpack Glance) home-screen widget ("Datapad") and its silent-freeze bug.
- `games-development.html` — case study: the university games-development portfolio (Unreal dissertation prototype, Unity group FPS, Unity platformer); video embeds instead of screenshots.
- `styles.css` — the only stylesheet; every page links it. Theming, layout, and all component styles (nav, cards, case blocks, gallery, lightbox) live here.
- `script.js` — tiny, single-purpose: wires up the lightbox click-to-zoom/Escape-to-close behavior for `.gallery img` elements. Nothing else.
- `screenshots/` — flat folder of 26 images referenced by the case-study pages (see convention above).

## Gotchas

- **Nav duplication**: updating the site nav (adding/renaming/reordering a project) means hand-editing the same markup block in all 7 HTML files — there's no templating. See "Shared nav" above.
- **`index.html`'s meta description is already stale relative to the nav**: it reads *"a live multi-broker trading system, a shipped desktop installer, a mobile companion app, and games development work"* and doesn't mention docs-assistant or the widget, even though both have their own nav links and case-study pages. Worth fixing next time that file is touched, and worth double-checking other pages' `<meta name="description">` for the same drift whenever a case study's content changes materially — these descriptions are hand-written per page and don't auto-update.
- **`<code>` vs `<strong>`** in `.log` entries: `<strong>` wraps emphasized phrases/findings in prose (e.g. "a **packaging defect**..."); `<code>` is reserved for literal technical tokens — function/class/endpoint names, filenames, exact strings (e.g. `Column`, `adb logcat`, `provider="claude"`, `FastMCP` → `MCPServer`, `=== date: title ===`). Keep that distinction when adding new log entries.
- No README or other notes file exists in this repo currently — this CLAUDE.md is the only map.
- **Nothing here updates itself when a project ships something.** Each case-study page is hand-written about a sibling repo (`../backtester`, `../sentrywatch`, `../Mobile_App`, `docs-assistant`); `../CLAUDE_NOTES.txt` is the workspace index of those projects and where each one's own map and history live.
