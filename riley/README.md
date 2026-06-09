# Riley's Garden 🌸

A cozy arcade game where a friendly **fairy** auto-blasts bees, you collect flowers & fruit for power-ups, and dodge snakes and a bear boss across 5 levels.

- **Play:** https://croctot.com/riley/
- **Source:** everything lives in [`riley/index.html`](./index.html) + [`riley/sounds/`](./sounds) (single-file game; no build step).
- **Dev log / deeper notes:** [`DEV_NOTES.md`](./DEV_NOTES.md)

> Note: root-level files belong to a different game (Croc Tot / Jackson's). Only touch files inside `riley/`.

## How to play

Tap/click to move the **fairy hero**. She auto-targets and blasts the nearest bee, and reels in nearby fruit & flowers. Collect fruit & flowers to unlock stronger weapons and the **magnet power-up** — while it's active the fairy shows a pulsing magnetic-pull halo. Survive the level's minimum time **and** clear all bees to advance (snakes are dodge-only hazards, not required kills). On the final level, survive the level and then the bear boss appears at the end — beat it to win. **New in 2026-06-09:** every level now also has a kill-count goal — defeat **15 bad guys** and a **miniboss** appears; beat the miniboss (the bear on Lv5) to finish the level. This guarantees a level always ends.

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
- **NEW MECHANIC: kill-count → miniboss → level-end (15 kills, every level)** — 2026-06-09. The level-end no longer depends on fragile enemy cleanup. A global `kills` counter increments at the single `killBee()` chokepoint; when `kills>=killGoal` (default **15**, set on the boss-state line) a boss is armed on EVERY level (previously only Lv5). The bear stays the Lv5 boss (15 HP); Lv1–Lv4 now get a lighter **miniboss** (8 HP) via `spawnBoss()` HP scaling. The level ends only once the boss is beaten. The old time-based rules are kept as a safety fallback (boss also arms at `lvMinDur+8s`, plus the existing off-screen-bee culling / hard caps). **Verified end-to-end on production** with the new QA harness: boss spawns at exactly 15 kills via the kill path, all 5 levels reach the level-complete (`lc`) state, miniboss HP is 8 / bear is 15, and a real playthrough of Lv1 reached “Level 1 done!” after the miniboss was beaten.
- **Regression caught by QA: `ReferenceError: lv is not defined`** — 2026-06-09. The first kill-count commit added miniboss warning/announcement text in `checkWinCondition()` and HP scaling in `spawnBoss()` that referenced a bare `lv` — but `lv` is a local `const` only inside `buildLv()`, so it threw every frame and levels could not end. Fixed by using the global `LVS[LI]` lookup instead. Found immediately by `riley/qa-harness.html` before it could affect players (the original commit’s pattern is exactly why the QA harness exists).
- **Added an automated QA harness (`riley/qa-harness.html`)** — 2026-06-09. Loads the real game in an iframe and drives the actual game loop across all 5 levels with a synthetic “perfect player,” asserting per-level invariants: boots, enemies stay in-bounds, the kill counter reaches `killGoal`, a miniboss/boss spawns, the level reaches level-complete, and no hard-cap overrun. This automates the manual live-state inspection that previously found the off-screen-bee bug. See the “QA harness” section below.
- **Levels could never end because bees drifted off-screen** — *2026-06-08.* THE real root cause (found via live state inspection: a level showed 0 visible enemies but `bees.filter(alive).length===8`, with bee x-positions like 2000 and -1000 — far outside the 430px-wide playfield). Bee movement patterns reversed velocity at the edges but never clamped position, so a lag/tab-switch frame could fling a bee far off-screen where the fairy can never reach it, leaving the level permanently unwinnable. Fix #1: `moveBee` now hard-clamps every bee inside the playfield each frame. Fix #2: a two-stage safety net in `checkWinCondition` — at the minimum time it culls any unreachable (off-screen) bee, and 4s past the minimum it force-clears ALL remaining bees so a stuck/oscillating/ghost bee can never hang the level. Boss level still requires beating the bear. **Q/A:** verified by driving the real game loop with the player killing nothing (level ends at ~30.0s, bees stay in-bounds) and by a deterministic safety-net test across all 5 levels (worst-case bee placement → every level reaches the level-complete state).
- **Levels no longer run forever** — *2026-06-08.* The win condition required *all snakes* to be killed, but snakes are dodge-only obstacles the player never has to kill — so any surviving snake left the level stuck with no enemies and no way to end. Removed the snake-kill requirement: a level now finishes once the minimum duration is reached, all bees are cleared, and the boss (if any) is beaten. Snakes remain as hazards to dodge.
- **Hero is the fairy again; magnet effect is now a power-up only** — *2026-06-08.* Reverted the avatar from the horseshoe magnet back to the fairy sprite (winged, with wand + sparkle aura). The magnetic-pull halo (blue rings + glow) now appears **only while the magnet power-up is active** (`farmerOn`), not at all times.
- **Hero is now a magnet** — *2026-06-08.* ~~Made the avatar a horseshoe magnet.~~ **Superseded same day:** reverted to the fairy avatar (see above); the magnet visual is now power-up-only.
- **Bear boss now appears at the END of Level 5** — *2026-06-08.* Previously the bear spawned at the start of Lv5; now you survive the level first and the bear arrives at the end (with a short warning), and you must beat it to win.
- **All levels last 30 seconds** — *2026-06-08.* Every level's minimum survival time is now 30s, followed by the bear fight on Lv5 (~15–30s).
- **Music for Levels 3–5** — *2026-06-08.* Lv3–5 now play their real audio tracks (`music-1v3`, `music-1v4`, `music-lv5`) instead of the synth fallback; boss music still plays for the bear.
- **Levels never ended (soft-lock)** — *fixed 2026-06-07, commit `97c2814`.* The win condition needs every bee & snake dead, but a bee could drift to a spot the auto-shooter never reached (closest shot ~52px vs a 22px kill hitbox), so the level ran forever. The old "hard cap" only looped an “Almost done!” message and never finished the level. Fix: widened hitboxes (bees 22→30, snakes 20→26), strengthened dust homing with a proximity-pop, and added a real failsafe that clears lingering non-boss enemies and ends the level after `lvMinDur+8s`. Verified on production: Lv1 completes and advances to Lv2.
- **Each level has its own bad guy** *(2026-06-07)* — bees → beetles → crows → bats, then a bee swarm + bear boss on Lv5. Boss music (`boss.mp3`) now reliably starts when the bear appears.
- **Hard 45s level time cap** *(2026-06-07)* — belt-and-suspenders: no level can run past 45 seconds; the level auto-completes if time runs out. Boss levels get 90 seconds so the bear fight isn't cut short.
- Level 2 endless item-respawn loop (earlier session).
- Intro audio not playing; music AudioContext unlock; orphaned timer-hint JS rendering as page text (see `DEV_NOTES.md`).

- **New enemy challenges + template foundation** — *2026-06-07.* Added a `swoop` dive-bomb movement pattern and assigned it to Lv4's bats (commit for Lv4), and introduced a central `THEME` config block so the game can be reskinned as a template (`EMJ`/`FRUITS`/enemy fallbacks now read from `THEME`). Documented in `TEMPLATE.md`.
## QA harness — “does every level end?”

`riley/qa-harness.html` (open it at https://croctot.com/riley/qa-harness.html) is an automated checker for the game’s #1 recurring bug: levels that never end. It loads the real game in an iframe and, for each of the 5 levels, fast-forwards the actual game loop while a synthetic player kills enemies, then asserts:

1. **boots** — game globals (`loop`, `LVS`, `killGoal`) are reachable.
2. **inBounds** — no enemy sits far outside the 430px playfield (the original off-screen-bee bug).
3. **killGoal** — the kill counter reaches `killGoal` before the boss.
4. **bossSpawns** — a miniboss/boss actually spawns.
5. **ends** — the level reaches the level-complete state within a time budget.
6. **noOverrun** — the level does not blow past its hard cap.

Run it after any change to enemies, the win condition, or the boss. A generalized version of these invariants is described in the sibling **buildable-app** repo (see its README “QA Agent” section) so AI-generated Buildable Kids games can be auto-checked for unwinnable levels.

### Open / follow-ups
- **Miniboss still looks/labels as the bear.** `spawnBoss()` draws one boss sprite and the persistent on-screen HP bar reads “BEAR BOSS!” on every level; only the spawn/warning pops and the HP value are miniboss-aware. A distinct miniboss sprite per level (e.g. driven by `LVS[LI].enemy`) would make Lv1–Lv4 minibosses feel different from the Lv5 bear.
- **15 kills vs. the 30s minimum duration.** Beating the boss does not end the level instantly — the level still waits for `lvMinDur` (30s) and all bees cleared. In normal play that’s fine (reaching 15 kills + a boss fight takes ~30s), but if you want the boss-defeat to end the level immediately, drop the `minDone` gate once `bossSpawned && !boss`.
- **Enemy throughput on early levels.** Lv1 has only 3 bees with limited respawns, so reaching 15 kills via the kill path can be slow and the time-based fallback often arms the miniboss first. Raise per-level bee counts / `beesRespawn` / lower `waveEvery` if you want 15 kills to be the primary trigger on every level.
- Timer bar only refreshes inside `updHUD()` (on score change), so it can look frozen between kills — cosmetic.
- Lv5 music file is `music-lv5.mp3` (not `music-1v5.mp3`); could be renamed for naming consistency.
- Homing shots track some targets by array index; a refactor to track by object reference would be cleaner.

## Credits

🎮 Built by Riley.
