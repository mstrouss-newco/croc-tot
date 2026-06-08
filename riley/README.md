# Riley's Garden 🌸

A cozy arcade game where a friendly **magnet** auto-blasts bees, you collect flowers & fruit for power-ups, and dodge snakes and a bear boss across 5 levels.

- **Play:** https://croctot.com/riley/
- **Source:** everything lives in [`riley/index.html`](./index.html) + [`riley/sounds/`](./sounds) (single-file game; no build step).
- **Dev log / deeper notes:** [`DEV_NOTES.md`](./DEV_NOTES.md)

> Note: root-level files belong to a different game (Croc Tot / Jackson's). Only touch files inside `riley/`.

## How to play

Tap/click to move the **magnet hero**. It auto-targets and blasts the nearest bee/snake, and while the magnet power-up is active it shows a pulsing magnetic-pull halo and reels in nearby fruit & flowers. Collect fruit & flowers to unlock stronger weapons. Survive the level's minimum time **and** clear all bees & snakes to advance. On the final level, survive the level and then the bear boss appears at the end — beat it to win.

## Levels

| # | Name | Bad guy | Theme |
|---|------|---------|-------|
| 1 | Sunny Garden | 🐝 Bees | bright & simple intro |
| 2 | Apple Orchard | 🪲 Beetles | faster, patrolling, they respawn |
| 3 | Rainy Garden | 🐦 Crows | wind + rain, erratic flight |
| 4 | Nighttime Garden | 🦇 Bats | dark night — bats now **dive-bomb** the player (swoop) |
| 5 | The Big Beehive | 🐝 Bees, then 🐻 Bear Boss | survive 30s, then the bear appears — beat it (with boss music!) to win |

Each level also has its own **enemy movement pattern** (`bp` in the `LVS` config): `linear`, `patrol`, `random`, `zigzag`, `swoop` (dive-bomb), and `swirl`.

## Reuse as a game template

This game is built to be **reskinned** into a different game without touching the engine. All game-wide identity (name, hero character, item graphics, sound folder) lives in a single `THEME` object at the top of the script, and each level's look & difficulty lives in the `LVS` array. Swap those two blocks (plus the files in `sounds/`) and you have a new game.

See **[`TEMPLATE.md`](./TEMPLATE.md)** for the full customization guide.

**Powering a game maker:** the design patterns from this game (the THEME/LVS split, the named enemy movement patterns, power-ups, win conditions and difficulty curve) have been distilled into a reusable mechanics library used by the sibling project **buildable-app** (an AI kids' game generator). See `MECHANICS.md` in that repo. Riley is the proven reference implementation those patterns come from.

## Build / deploy

No build tooling — `riley/index.html` is a self-contained HTML/JS/CSS file. Committing to `main` deploys automatically to `croctot.com/riley/` (allow ~1 min for the CDN to update; append a `?v=` query string to bust cache when testing).

Edit the file directly on GitHub; per `DEV_NOTES.md`, the CodeMirror instance is reachable at `document.querySelector('.cm-content').cmTile.view` for programmatic edits.

## Known bugs & fixes

### Fixed
- **Hero is now a magnet** — *2026-06-08.* The player character is a red horseshoe magnet (replacing the fairy sprite) and shows an animated magnetic-pull halo while the magnet power-up is active.
- **Bear boss now appears at the END of Level 5** — *2026-06-08.* Previously the bear spawned at the start of Lv5; now you survive the level first and the bear arrives at the end (with a short warning), and you must beat it to win.
- **All levels last 30 seconds** — *2026-06-08.* Every level's minimum survival time is now 30s, followed by the bear fight on Lv5 (~15–30s).
- **Music for Levels 3–5** — *2026-06-08.* Lv3–5 now play their real audio tracks (`music-1v3`, `music-1v4`, `music-lv5`) instead of the synth fallback; boss music still plays for the bear.
- **Levels never ended (soft-lock)** — *fixed 2026-06-07, commit `97c2814`.* The win condition needs every bee & snake dead, but a bee could drift to a spot the auto-shooter never reached (closest shot ~52px vs a 22px kill hitbox), so the level ran forever. The old "hard cap" only looped an “Almost done!” message and never finished the level. Fix: widened hitboxes (bees 22→30, snakes 20→26), strengthened dust homing with a proximity-pop, and added a real failsafe that clears lingering non-boss enemies and ends the level after `lvMinDur+8s`. Verified on production: Lv1 completes and advances to Lv2.
- **Each level has its own bad guy** *(2026-06-07)* — bees → beetles → crows → bats, then a bee swarm + bear boss on Lv5. Boss music (`boss.mp3`) now reliably starts when the bear appears.
- **Hard 45s level time cap** *(2026-06-07)* — belt-and-suspenders: no level can run past 45 seconds; the level auto-completes if time runs out. Boss levels get 90 seconds so the bear fight isn't cut short.
- Level 2 endless item-respawn loop (earlier session).
- Intro audio not playing; music AudioContext unlock; orphaned timer-hint JS rendering as page text (see `DEV_NOTES.md`).

- **New enemy challenges + template foundation** — *2026-06-07.* Added a `swoop` dive-bomb movement pattern and assigned it to Lv4's bats (commit for Lv4), and introduced a central `THEME` config block so the game can be reskinned as a template (`EMJ`/`FRUITS`/enemy fallbacks now read from `THEME`). Documented in `TEMPLATE.md`.
### Open / follow-ups
- Timer bar only refreshes inside `updHUD()` (on score change), so it can look frozen between kills — cosmetic.
- Lv5 music file is `music-lv5.mp3` (not `music-1v5.mp3`); could be renamed for naming consistency.
- Homing shots track some targets by array index; a refactor to track by object reference would be cleaner.

## Credits

🎮 Built by Riley.
