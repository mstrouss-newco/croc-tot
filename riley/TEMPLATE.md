# Game Template Guide

This folder is a self-contained 2D "auto-blaster" arcade game that runs as a single
HTML file. It was built as *Riley's Garden*, but it is designed to be **reskinned**
into a completely different game (new background, characters, items and sounds)
**without touching the engine code**.

This guide explains how the game is structured and how to make your own version.

---

## 1. How to make your own game from this template

1. Copy `index.html` and the `sounds/` folder into a new folder.
2. Open `index.html` and find the **`THEME`** block near the top of the `<script>`.
3. Change the values in `THEME` to rename the game and swap the item graphics.
4. Edit the **`LVS`** array (right below `THEME`) to change each level's look,
   enemies and difficulty.
5. Drop your own audio files into `sounds/` (keep the same file names).
6. Open the file in a browser to play. That's it — no build step.

Everything a designer normally wants to change lives in just **two config blocks**
(`THEME` + `LVS`). The rest of the file is the reusable engine.

---

## 2. The two config blocks

### `THEME` — game-wide identity (the "skin")

```js
const THEME = {
  name:    "Riley's Garden",      // branding / page title
  tagline: 'Pick flowers & fruit, dodge the bugs!',
  hero:    { emoji:'🧚', label:'fairy' },  // the auto-blasting helper
  items:   { sunflower:'🌻', strawberry:'🍓', apple:'🍎', /* ... */ },
  fruits:  ['strawberry','apple','blueberry','grapes','orange'],
  sounds:  'sounds/',             // folder with music + sfx
  enemyFallback:{ nm:'bee', em:'🐝', mad:'😡', wing:1, wc:'#e0f7ff' }
};
```

| Field | What it controls |
|-------|------------------|
| `name` / `tagline` | Title-screen branding |
| `hero` | The character the player moves around |
| `items` | The collectible graphics (emojis or labels) |
| `fruits` | Which items count as edible power-ups |
| `sounds` | Folder the audio is loaded from |
| `enemyFallback` | Default enemy used if a level omits its own |

### `LVS` — per-level skin & difficulty

Each entry in the `LVS` array is one level. The most useful fields:

| Field | Meaning |
|-------|---------|
| `nm` | Level name |
| `bgT` / `bgB` | Sky gradient top / bottom colour |
| `gc` / `gr` | Ground colours |
| `items` | Which collectibles spawn `{t:'apple', n:4}` |
| `enemy` | This level's bad guy `{nm, em, mad, wing, wc}` |
| `bees` / `snk` | How many enemies / snakes |
| `bs` / `ss` | Enemy / snake speed |
| `bp` | **Movement pattern** (see below) |
| `minDur` | Minimum level length (ms) |
| `tip` | Hint shown to the player |

---

## 3. Enemy movement patterns (`bp`)

The engine ships with several reusable movement behaviours. Set `bp` on a level
to pick one — no code needed:

| `bp` value | Behaviour |
|------------|-----------|
| `linear` | Drifts straight across, gentle bob |
| `patrol`  | Marches back and forth, steps down at edges |
| `random`  | Wanders unpredictably |
| `zigzag`  | Sine-wave weaving |
| `swoop`   | Drifts, then periodically **dive-bombs** toward the player |
| `swirl`   | Orbits the centre of the screen |

To add a brand-new behaviour, add one `else if (b.pat==='yourname'){ ... }` branch
inside `moveBee()` and reference it with `bp:'yourname'`.

---

## 4. Swapping graphics

This template uses **emoji** as sprites by default, which means you can reskin
characters and items just by changing a single emoji in `THEME.items` or a
level's `enemy.em`. No image files required.

If you want **real images** instead of emoji, that is the one place that needs
engine work: the draw functions (`drawBee`, the player draw code) would be
extended to draw an `Image` when a sprite URL is provided. The config is already
structured so each enemy/item is one object — a future upgrade can add a
`sprite:'cat.png'` field that the renderer prefers over the emoji.

---

## 5. Sounds

Audio lives in the folder named by `THEME.sounds` (default `sounds/`).
Keep the existing file names so the engine finds them:

- `music-1v1.mp3` … `music-1v5.mp3` — per-level background music
  (note: the project uses **`1v`**, not `lv`, in these names)
- `boss.mp3` — plays during the boss fight

Audio only starts after the first tap/click (browser autoplay rule), which the
engine already handles by unlocking the audio context on the first user gesture.

---

## 6. What stays fixed (the engine)

Everything else — the game loop, collision, scoring, HUD, level flow, the auto-
blaster, the timer cap, boss logic — is shared engine code you normally do **not**
edit when making a new skin. Keeping game logic separate from `THEME`/`LVS` is
what makes this folder reusable as a template.
