# croc-tot repository

This repo hosts **two separate browser games**. They are deployed together via GitHub Pages on the custom domain **croctot.com** (see `CNAME`).

## ⚠️ Read this first — two games, do not mix them up

| Game | Owner | Lives in | Served at |
|------|-------|----------|-----------|
| **Croc Tot** | Jackson | root `index.html` (and `croc-tot.html` as a backup copy) | **https://croctot.com/** |
| **Riley's Garden** | Riley | `riley/index.html` + `riley/sounds/` | **https://croctot.com/riley/** |

Root `index.html` is **Croc Tot**. The site root (croctot.com) must always show Croc Tot.

**The recurring bug to avoid:** Riley's Garden kept getting written into the root `index.html`, which overwrote Croc Tot on croctot.com. Riley's Garden belongs **only** in the `riley/` folder. Never put Riley's Garden (or any non–Croc Tot game) at the repo root.

## Where things live

- `index.html` — Croc Tot (Jackson's game). This is what croctot.com serves.
- `croc-tot.html` — backup/source copy of the Croc Tot game.
- `install.html` — install / PWA landing page.
- `CNAME` — custom domain (`croctot.com`).
- `riley/` — Riley's Garden (Riley's game) + its docs (`riley/README.md`, `riley/DEV_NOTES.md`, `riley/TEMPLATE.md`) and `riley/sounds/`.

## Deploy

No build step. Both games are self-contained HTML files. Committing to `main` deploys automatically to croctot.com (allow ~1 min for the CDN; add a `?v=` query string to bust cache when testing).

## Change log

Newest first. **Every code/content change must be logged here** (see AGENTS.md).

### 2026-06-08 — Restored Croc Tot to the site root
- Root `index.html` had become Riley's Garden (it was renamed from `rileys_game_updated.html`), which overwrote Jackson's Croc Tot on croctot.com.
- Copied the latest Croc Tot game (from `croc-tot.html`) back into root `index.html` so croctot.com serves Croc Tot again.
- Riley's Garden is unchanged and still lives in `riley/` (served at croctot.com/riley/).
- Added this README and `AGENTS.md` documenting the two-game layout so the root index stops getting overwritten.
