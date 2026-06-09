# Riley's Garden — Dev Notes

## Off-screen enemies/boss + magnet sound fix (2026-06-09)

**Bug (reported with screenshots):** Bad guys wandered off the visible play area leaving the screen empty, and when the boss finally appeared it charged straight off-screen too. The magnet ("farmer") power-up also played an annoying activation sound.

**Root cause:** In `moveBee()` several movement patterns let enemies leave the screen — `linear` and `zigzag` deliberately *wrapped* with a ±40px off-screen buffer, and `swirl`/`swoop`/`random` set position via trig or reversed velocity at the edge without clamping the position, so fast movers overshot. `updateBoss()` had no bounds at all during its charge-at-player state, so it followed the fairy off the playfield.

**Fix (commit `829cc87`):**
- Added a universal HARD ON-SCREEN CLAMP at the end of `moveBee()` (runs after every pattern): clamps `x` to `[24, W-24]` and `y` to `[40, H-GH()-20]`, and flips any velocity component that points outward so enemies bounce back inward instead of escaping.
- Added the equivalent HARD ON-SCREEN CLAMP at the end of `updateBoss()`: clamps boss `x` to `[40, W-40]` and `y` to `[60, H-GH()-30]`, flipping `vx` inward — so the boss can never charge off the playfield.
- Removed the magnet activation sound by deleting the `sfx('farmer')` call in `loop()`. The magnet power-up itself is unchanged and still works.

**Verification (live, croctot.com/riley/):** Stress-tested all 6 bee patterns (`linear/patrol/random/zigzag/swirl/swoop`) at 18px/frame for 229 samples → 0 off-screen excursions. Pushed the boss to 6 extreme off-screen positions (edges + both diagonal corners) → all clamped back in bounds after one `updateBoss()` tick. Confirmed `sfx('farmer')` is gone from the deployed bundle.

## Kill-count → miniboss → level-end mechanic (2026-06-09)

This is the deterministic level-end mechanic. Implementation map in `riley/index.html`:

- **State:** `kills` and `killGoal` (default `15`) are globals declared on the boss-state line (`let ...,kills=0,killGoal=15,bossPending=false,bossSpawned=false...`).
- **Counting:** `kills++` runs at the END of `killBee(i,x,y)` — the single chokepoint where any bad guy dies. Snakes are dodge-only and intentionally don’t count.
- **Reset:** `kills=0` in `buildLv()` (and `startLv()` rebuilds the level), so each level starts fresh.
- **Arming the boss on every level:** in `buildLv()` the level now sets `bossPending=true` for ALL levels (previously only `lv.bearBoss` did). `lv.bearBoss` still distinguishes the Lv5 bear from the Lv1–Lv4 minibosses.
- **Spawn trigger:** in `checkWinCondition()`, the boss spawns when `kills>=killGoal` OR (as a safety fallback) `minDone && lvTimer>=lvMinDur+8000`. The warning/announcement pops are miniboss-aware (`LVS[LI].bearBoss ? 'BEAR' : 'MINIBOSS'`).
- **Boss HP:** `spawnBoss()` scales HP — `LVS[LI].bearBoss ? 15 : 8` (bear vs. miniboss).
- **Defeat → end:** `hitBossWep(dmg)` sets `boss=null` when `bossHp<=0`; `checkWinCondition()` then finishes the level once `allBeesGone && bossGone && minDone`.

### Gotcha that bit us (and how QA caught it)
`lv` is a **local `const` inside `buildLv()` only** — it is NOT a global. The first version referenced `lv.bearBoss` inside `checkWinCondition()` and `spawnBoss()`, which threw `ReferenceError: lv is not defined` every frame and made levels unwinnable. **Always use `LVS[LI]` to read the current level config outside `buildLv()`.** This was caught immediately by `riley/qa-harness.html` (drives the real loop across all 5 levels and asserts each reaches the level-complete state), not by a player.

### Tuning knobs
- `killGoal` (boss-state line) — kills needed before the miniboss (currently 15).
- `spawnBoss()` HP values — miniboss `8`, bear `15`.
- The `lvMinDur+8000` fallback — lower it if you want the boss to never depend on the timer.
- Per-level enemy counts / `beesRespawn` / `waveEvery` in `LVS` — raise enemy throughput so 15 kills is reached before the time fallback on early levels.



### Miniboss visuals, instant level-end & wave tuning (2026-06-09)

Three refinements on top of the base mechanic:

- **Distinct miniboss sprite.** `spawnBoss()` now tags the boss object with `mini` (true for Lv1–Lv4) and `em`/`wc` from `LVS[LI].enemy`. `drawBoss()` early-returns into a new **`drawMiniBoss(b)`** helper when `b.mini` is set, which draws a giant `b.em` emoji + a 👑 crown + an angry radial aura + a floating HP bar (canvas `roundRect`). The elaborate hand-drawn bear vector art in `drawBoss()` is untouched and used only for the Lv5 bear. The HP-bar label (`#bblbl`) is set in `updHUD()` to `boss.mini ? em+' MINIBOSS!' : '🐻 BEAR BOSS!'`.
- **Instant level-end on boss defeat.** `checkWinCondition()` now does `if(bossSpawned && bossGone) finishLevel();` so beating the (mini)boss ends the level immediately, without waiting out `lvMinDur`. The old `allBeesGone && bossGone && minDone` rule is the fallback for the pre-boss phase.
- **Wave tuning.** Waves spawn every `waveEvery` ms for the whole `lvMinDur`; with the old 8–11s spacing you couldn’t reach 15 kills in 30s. Set `waveEvery:3500` and `beesRespawn:3` on all levels so the kill-count path (not the time fallback) is the primary boss trigger.

### Another scoping gotcha (QA harness edition)
`LI` (current level index) is **closure-scoped**, like `lv` — you can’t set it from outside via `window.LI`. To drive a specific level (in the QA harness or the console) call the game’s own **`startGame(idx)`**, which sets `LI` and builds the level. The QA harness was briefly reporting every boss as Lv1’s bee because it set `window.lvIdx` instead; fixed to use `startGame(idx)`, and it now also asserts the miniboss emoji matches `LVS[idx].enemy.em` and that Lv5 is the bear.

## Repository
- Repo: mstrouss-newco/croc-tot
- Riley's game lives entirely in: `riley/index.html` + `riley/sounds/`
- **DO NOT touch root files** — those belong to Croc Tot / Jackson's game at croctot.com
- Production URL: https://croctot.com/riley/

## File Naming Conventions
- Main file: must be exactly `index.html` inside `riley/` folder
- Audio files: `riley/sounds/music-1v1.mp3`, `music-1v2.mp3`, etc. (note: `1v` not `lv`)
- Boss music: `riley/sounds/boss.mp3` (one file used for all boss levels)

## Audio Architecture
- **Intro/welcome clip**: base64-encoded M4A embedded directly in `<audio id="introAudio">` inside index.html — this is Riley's voice saying "welcome to Riley's garden". Plays on "Let's Play!" tap via `startIntro()`.
- **Level music (Lv1)**: `riley/sounds/music-1v1.mp3` — real audio file
- **Level music (Lv2)**: `riley/sounds/music-1v2.mp3` — real audio file
- **Levels 3–5**: synth fallback (no audio files uploaded yet)
- **Boss music**: `riley/sounds/boss.mp3` — plays on Bear Boss spawn, stops on death
- AudioContext must be unlocked via `resumeAC()` before any audio plays — call this from user gesture handlers

## CodeMirror 6 Editor (GitHub)
- Access the editor instance via: `document.querySelector('.cm-content').cmTile.view`
- Use `view.state` and `view.dispatch()` to read/write file content programmatically
- Batch all changes in a single `dispatch()` call with a `changes` array

## Pending / To Do
- [x] Lv3–5 real music wired (2026-06-08): `music-1v3.mp3`, `music-1v4.mp3` present; Lv5 uses `music-lv5.mp3` (the uploaded filename). Optional: rename `music-lv5.mp3` → `music-1v5.mp3` for naming consistency and update the `realTracks` entry to match.
- [ ] Verify welcome clip sounds correct on real device (Riley's voice)
- [ ] Clean up empty `riley/sounds/boss/` subfolder (unused)

## Session Log

### 2026-06-08 — magnet hero, boss-at-end, 30s levels, Lv3-5 music
- **Hero is now a MAGNET.** Rewrote `drawFairy(x,y,fr,iv)` to draw a red horseshoe magnet (silver pole tips, friendly eyes) instead of the fairy. While the magnet power-up is active (`farmerOn`), an animated **magnetic-pull halo** (3 expanding blue rings + radial glow) pulses around the sprite. Set `THEME.hero` to the magnet emoji/label. Disabled the now-redundant separate `drawFarmer` overlay call and recoloured the pull-range ring blue (`#6eaaff`). Updated the intro subtitle text from "The fairy auto-blasts bees" to "Your magnet auto-blasts bees". Note: the projectile weapon is still called "Fairy Dust" (a separate game element, left unchanged).
- **Bear boss now appears at the END of Lv5, not the start.** Removed `if(lv.bearBoss)spawnBoss(gh)` from `buildLv()` (it spawned the bear immediately). Added `bossPending/bossSpawned/bossWarned` state. In `checkWinCondition()`: on a boss level the bear is spawned once `lvTimer>=lvMinDur`, with a "Bear is coming!" warning ~3s before; the level will NOT finish while the boss is pending or alive — you must beat the bear to win. Made the time-up auto-clear and the `lvMinDur+8000` failsafe boss-aware (`_failAt = bossPending ? lvMinDur+35000 : lvMinDur+8000`) and added `&&!bossPending` to the all-cleared finish check, so the bear fight (≈15-30s, HP 15) isn't cut short.
- **Levels are now 30s.** Set `minDur:30000` on all 5 levels (were 30/33/36/40/45s). Boss level = 30s of survival, then the bear fight.
- **Music for Lv3-5 wired up.** Extended `realTracks` from 2 to 5 entries: `sounds/music-1v1.mp3` … `music-1v4.mp3`, and `sounds/music-lv5.mp3` for Lv5 (kept the on-disk `lv5` filename rather than the `1v` convention so it matches the uploaded file — the previous code only listed 2 tracks so Lv3-5 fell back to synth). `startMusic(LI)` indexes `realTracks[LI]`, so each level now plays its real track; boss music (`boss.mp3`) still triggers on bear spawn.
- **Per-level variety confirmed:** distinct movement patterns linear/patrol/random/swoop/swirl across Lv1-5, plus distinct environments (clear / orchard+respawn / rain+wind / night+stars+ghost-bees / bear boss) and enemies (bee/beetle/crow/bat/bee+bear).
- QA: tested on the deployed site (croctot.com/riley/?v=). Verified Lv5 has `boss===null` at start, the bear spawns exactly at lvTimer=30000, the level stays in play while the boss is alive, and finishes (`GS='lc'`) only after the boss is killed. Magnet + halo render confirmed on screen. Audio can't be auto-verified (needs a real tap) — confirm Lv3-5 + boss music on a real device.
- Commits: 1b02e3c (main update), e1d29b6 (intro text).

### 2026-06-07 (later) — new enemy challenge + template foundation
- **Lv4 dive-bombing bats:** added a new `swoop` movement pattern to `moveBee()` (drifts across, then periodically dives toward the player's row and climbs back). Switched Lv4 `bp:'zigzag'` → `bp:'swoop'` and updated its tip. Verified live on Lv4 (bat y-positions sweep top↔bottom).
- **THEME config block:** added a single `THEME` object above `LVS` holding game-wide identity — `name`, `tagline`, `hero`, `items`, `fruits`, `sounds` folder, and `enemyFallback`. Rewired `EMJ = THEME.items`, `FRUITS = new Set(THEME.fruits)`, and the drawBee/updHUD enemy fallbacks to read `THEME.enemyFallback`. Goal: reskin the game by editing only `THEME` + `LVS`, not engine code.
- **TEMPLATE.md:** new guide explaining how to clone this folder into a different game (swap THEME/LVS, drop new files in sounds/). Documents the 6 movement patterns (linear/patrol/random/zigzag/swoop/swirl) and the emoji-sprite approach.
- README updated with the swoop bats, pattern list, and a 'Reuse as a template' section.
- Note: `THEME` is a `const` inside the script closure, so it is intentionally NOT on `window`; verify deploys by fetching the source text, not `window.THEME`.

### 2026-06-07 (later) — per-level enemies + boss music
- **Distinct bad guy per level.** Added a per-level `enemy` config object: `{nm, em, mad, wing, wc}` (name, emoji, angry-emoji, draw-insect-wings flag, wing colour). Lineup: Lv1 🐝 bees, Lv2 🪲 beetles, Lv3 🐦 crows, Lv4 🦇 bats, Lv5 🐝 bee swarm + 🐻 bear boss. Reuses the existing bee movement/collision — only the look + HUD/messages change.
- Wiring: `drawBee` reads `LVS[LI].enemy` (wings drawn only when `wing`), the HUD `×N` counter uses `enemy.em`, the “now clear the …” pop uses `enemy.nm`, and Lv2–Lv4 `tip` text was reworded (beetles/crows/bats).
- **Boss music hardened.** `startBossMusic()` now calls `resumeAC()` and, if the browser blocks autoplay, keeps `_bossAudio` and retries on the next `pointerdown`/`touchstart` instead of nulling it on first failure (the old `.play().catch(()=>_bossAudio=null)` could permanently kill boss music if the first attempt wasn’t a trusted gesture). `boss.mp3` already triggers via `spawnBoss()` on Lv5; this just makes it reliable.
- QA: verified on production each level shows its creature (HUD + sprites), and the bear boss + HP bar appear on Lv5. Note: boss music can’t be auto-verified from a scripted `startGame()` call because browsers require a real user gesture to start audio — confirm on a real tap.
- Commits: `faf3c59` (enemies), `b7d8c88` (boss music).

### 2026-06-07
- **Bug: levels never end (soft-lock).** Win condition needs ALL bees + snakes dead AND `lvTimer>lvMinDur`. A bee could drift to a spot the auto-shooter never reached — measured closest shot approach ~52px vs the 22px kill hitbox — so the last bee was un-killable and the level ran forever. The `lvDur+15000` "hard cap" only looped an `⏰ Almost done!` message + reset the timer; it never called `finishLevel()`.
- **Fixes (in `index.html`):**
  - Widened auto-shot hitboxes: bees `22 → 30`, snakes `20 → 26`.
  - Strengthened dust homing (`.04 → .12`, target speed `3.5 → 4.8`) + added a proximity-pop: a dust shot within 34px of its target kills it, resolved via `bees.indexOf(ti)` so the stale-index-after-filter targeting bug can't misfire.
  - Replaced the broken hard cap with a real failsafe: once `lvTimer>lvMinDur+8000`, force-clear any remaining non-boss enemies and call `finishLevel()`. Bosses still must be beaten (failsafe skips while `boss` is alive).
- **QA / verified on production** (`croctot.com/riley/`): played Lv1 end-to-end → `GS` reached `lc`, "Amazing! Level 1 done!" shown, "Next Level" advanced to `LI=1` (Lv2, 4 bees / 2 snakes). Timer bar now advances past 0% (previously frozen).
- **Known follow-ups:** timer bar only refreshes inside `updHUD()` (on score change), so it can look static between kills — cosmetic, not blocking. Homing shots still store `s.tgt.i` by index elsewhere; the new proximity-pop sidesteps it, but a fuller refactor to track targets by object ref would be cleaner.
- **Added absolute 45s level time cap** (commit pending). Belt-and-suspenders on top of the `lvMinDur+8000` failsafe: in the loop, `_maxDur=boss?90000:45000`; once `lvTimer>_maxDur` while `GS==='play'`, clear remaining enemies and `finishLevel()`. Boss levels get 90s so the bear fight isn't cut short; if even that elapses the boss is cleared and the level finishes (kept kid-friendly — no harsh game-over). Note: Lv5 `minDur` is 45000, which is why boss levels need the longer 90s cap. Verified the cap is deployed and normal Lv1→Lv2 progression still works.
- Commit: `97c2814`

### 2026-06-06
- Fixed intro audio: `startIntro()` existed but was never called — added call inside `stbtn` ("Let's Play!") click handler
- Fixed music AudioContext unlock: added `resumeAC()` call inside `.lvbtn` forEach handler before `startGame()`
- Fixed orphaned JS code: timer-hint logic (`if(lvTimer>=lvMinDur&&!_timesUpHinted)`) was accidentally placed before `<html>` tag (from a prior commit), causing it to render as visible text on desktop/tablet — moved it back inside the game loop
- Confirmed Level 1 music playing: `music-1v1.mp3`, `currentTime ~54s`, `volume 0.7`
- Commits: `098a8be`, `b4ae9bd`

### Prior Sessions
- Added level select screen with 5 level buttons (Lv1–Lv5) with emoji icons
- Wired in Lv1 and Lv2 real audio music
- Added boss music (`boss.mp3`) — plays on Bear Boss spawn, stops on death
- Fixed Level 2 endless loop bug (item respawn block)
- Added bee cap and wave spawner
