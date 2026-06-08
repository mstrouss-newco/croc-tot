# AGENTS.md — instructions for AI agents working in this repo

Read this **and** `README.md` before making any change.

## You can change anything — but log it

You are free to adjust, refactor, or improve anything in this repo. The only hard rule on process: **log every code/content change in `README.md`** under the "Change log" section (newest first, with the date and a short description of what you changed and why). Keep that log accurate so the next agent has context.

## This repo contains TWO separate games — do not mix them

| Game | Owner | Lives in | Served at |
|------|-------|----------|-----------|
| **Croc Tot** | Jackson | root `index.html` (backup copy in `croc-tot.html`) | **https://croctot.com/** |
| **Riley's Garden** | Riley | `riley/index.html` + `riley/sounds/` | **https://croctot.com/riley/** |

### The #1 mistake to avoid
Agents have repeatedly written **Riley's Garden into the root `index.html`**, which **overwrites Croc Tot on croctot.com**. Do not do this.

- Root `index.html` is **Croc Tot (Jackson's game)** and must stay Croc Tot. croctot.com must always serve Croc Tot.
- Riley's Garden belongs **only** inside `riley/`. Never copy, move, or "promote" it to the repo root.
- If you are working on Riley's Garden, edit `riley/index.html` and files under `riley/` only — see `riley/DEV_NOTES.md` and `riley/README.md`.
- If you are working on Croc Tot, edit root `index.html` (and keep `croc-tot.html` as the matching backup if you change the game).

## Deploy
No build step. Commit to `main` → auto-deploys to croctot.com via GitHub Pages (~1 min CDN delay; add `?v=` to bust cache while testing). Don't edit `CNAME` (it pins the domain to croctot.com).

## Quick checklist before you commit
1. Am I editing the right game's folder? (root = Croc Tot, `riley/` = Riley's Garden)
2. Did I avoid overwriting root `index.html` with anything other than Croc Tot?
3. Did I add an entry to the `README.md` change log describing my change?
