# Riley's Garden 🌸

A cozy arcade game where a fairy auto-blasts bees, you collect flowers & fruit for power-ups, and dodge snakes and a bear boss across 5 levels.

- **Play:** https://croctot.com/riley/
- **Source:** everything lives in [`riley/index.html`](./index.html) + [`riley/sounds/`](./sounds) (single-file game; no build step).
- **Dev log / deeper notes:** [`DEV_NOTES.md`](./DEV_NOTES.md)

> Note: root-level files belong to a different game (Croc Tot / Jackson's). Only touch files inside `riley/`.

## How to play

Tap/click to move the fairy. She auto-targets and blasts the nearest bee/snake. Collect fruit & flowers to unlock stronger weapons. Survive the level's minimum time **and** clear all bees & snakes to advance. Beat the bear boss on the final level.

## Levels

| # | Name | Bad guy | Theme |
|---|------|---------|-------|
| 1 | Sunny Garden | 🐝 Bees | bright & simple intro |
| 2 | Apple Orchard | 🪲 Beetles | faster, patrolling, they respawn |
| 3 | Rainy Garden | 🐦 Crows | wind + rain, erratic flight |
| 4 | Nighttime Garden | 🦇 Bats | dark, some hide like ghosts |
| 5 | The Big Beehive | 🐝 Bees + 🐻 Bear Boss | beat the bear (with boss music!) to win |

## Build / deploy

No build tooling — `riley/index.html` is a self-contained HTML/JS/CSS file. Committing to `main` deploys automatically to `croctot.com/riley/` (allow ~1 min for the CDN to update; append a `?v=` query string to bust cache when testing).

Edit the file directly on GitHub; per `DEV_NOTES.md`, the CodeMirror instance is reachable at `document.querySelector('.cm-content').cmTile.view` for programmatic edits.

## Known bugs & fixes

### Fixed
- **Levels never ended (soft-lock)** — *fixed 2026-06-07, commit `97c2814`.* The win condition needs every bee & snake dead, but a bee could drift to a spot the auto-shooter never reached (closest shot ~52px vs a 22px kill hitbox), so the level ran forever. The old "hard cap" only looped an “Almost done!” message and never finished the level. Fix: widened hitboxes (bees 22→30, snakes 20→26), strengthened dust homing with a proximity-pop, and added a real failsafe that clears lingering non-boss enemies and ends the level after `lvMinDur+8s`. Verified on production: Lv1 completes and advances to Lv2.
- **Each level has its own bad guy** *(2026-06-07)* — bees → beetles → crows → bats, then a bee swarm + bear boss on Lv5. Boss music (`boss.mp3`) now reliably starts when the bear appears.
- **Hard 45s level time cap** *(2026-06-07)* — belt-and-suspenders: no level can run past 45 seconds; the level auto-completes if time runs out. Boss levels get 90 seconds so the bear fight isn't cut short.
- Level 2 endless item-respawn loop (earlier session).
- Intro audio not playing; music AudioContext unlock; orphaned timer-hint JS rendering as page text (see `DEV_NOTES.md`).

### Open / follow-ups
- Timer bar only refreshes inside `updHUD()` (on score change), so it can look frozen between kills — cosmetic.
- Lv3–5 still use synth-fallback music; real `music-1v3/4/5.mp3` files not uploaded yet.
- Homing shots track some targets by array index; a refactor to track by object reference would be cleaner.

## Credits

🎮 Built by Riley.
