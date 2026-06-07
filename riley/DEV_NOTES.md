# Riley's Garden — Dev Notes

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
- [ ] Upload `riley/sounds/music-1v3.mp3`, `music-1v4.mp3`, `music-1v5.mp3` to replace synth fallback for Lv3–5
- [ ] Verify welcome clip sounds correct on real device (Riley's voice)
- [ ] Clean up empty `riley/sounds/boss/` subfolder (unused)

## Session Log

### 2026-06-07
- **Bug: levels never end (soft-lock).** Win condition needs ALL bees + snakes dead AND `lvTimer>lvMinDur`. A bee could drift to a spot the auto-shooter never reached — measured closest shot approach ~52px vs the 22px kill hitbox — so the last bee was un-killable and the level ran forever. The `lvDur+15000` "hard cap" only looped an `⏰ Almost done!` message + reset the timer; it never called `finishLevel()`.
- **Fixes (in `index.html`):**
  - Widened auto-shot hitboxes: bees `22 → 30`, snakes `20 → 26`.
  - Strengthened dust homing (`.04 → .12`, target speed `3.5 → 4.8`) + added a proximity-pop: a dust shot within 34px of its target kills it, resolved via `bees.indexOf(ti)` so the stale-index-after-filter targeting bug can't misfire.
  - Replaced the broken hard cap with a real failsafe: once `lvTimer>lvMinDur+8000`, force-clear any remaining non-boss enemies and call `finishLevel()`. Bosses still must be beaten (failsafe skips while `boss` is alive).
- **QA / verified on production** (`croctot.com/riley/`): played Lv1 end-to-end → `GS` reached `lc`, "Amazing! Level 1 done!" shown, "Next Level" advanced to `LI=1` (Lv2, 4 bees / 2 snakes). Timer bar now advances past 0% (previously frozen).
- **Known follow-ups:** timer bar only refreshes inside `updHUD()` (on score change), so it can look static between kills — cosmetic, not blocking. Homing shots still store `s.tgt.i` by index elsewhere; the new proximity-pop sidesteps it, but a fuller refactor to track targets by object ref would be cleaner.
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
