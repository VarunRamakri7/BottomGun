# BottomGun

BottomGun is a browser-based endless flyer: you steer a small airplane around a cylindrical “track”, collect cyan pickups for energy, and avoid red obstacles. Distance and level increase as you survive longer.

**[Play the live demo](https://varunramakri7.github.io/BottomGun/)**

---

## Tech stack

| Piece | Role |
|--------|------|
| **Three.js** (`scripts/three.min.js`) | WebGL scene, camera, fog, lights, shadows, meshes |
| **GSAP 3** (`scripts/gsap.min.js`) | Short tweens for particle bursts (coin/enemy effects) |
| **Vanilla JavaScript** (`scripts/game.js`) | Game logic, animation loop, DOM HUD updates |
| **HTML / CSS** | Layout, HUD, self-hosted **Playfair Display** (WOFF2 in `fonts/`) |

---

## Repository layout

```
index.html          # Page shell, HUD markup, script tags (order: GSAP → Three → game)
css/
  styles.css        # Font @font-face, global reset, body/link styles, .world base
  game.css          # Full-screen game shell, header, score, energy bar, messages
fonts/              # Playfair Display (Latin subset, WOFF2)
scripts/
  gsap.min.js       # Animation library (vendored)
  three.min.js      # Three.js (vendored; matches legacy geometry APIs in game.js)
  game.js           # All gameplay and rendering logic (see comments in file)
```

The inline SVG in `index.html` drives the **level progress ring** (`stroke-dashoffset` is updated from JavaScript as distance increases).

---

## How it works (implementation)

### Game loop

`game.js` uses `requestAnimationFrame` in `loop()`. Each frame it:

1. Computes **`deltaTime`** from wall-clock time (ms between frames).
2. Branches on **`game.status`** (`"intro"` → help overlay; `"paused"` → frozen frame; `"playing"` → flight; `"gameover"` → fall; `"waitingReplay"` → tap to restart).
3. While playing, checks **distance milestones** to spawn coins, ramp speed, spawn enemies, and level up.
4. Updates the plane, sea waves, clouds, coins, enemies, and particles, then **`renderer.render(scene, camera)`**.

### Simulation state

Almost all tunables live on a single object, **`game`**, reset in **`resetGame()`**: speed curves, energy drain, spawn intervals, level spacing, plane movement sensitivity, collision knockback, sea dimensions, etc. Adjusting difficulty or feel usually means editing those numbers (the top of `game.js` documents structure in comments).

### Pointer input

With **mouse or touch**, the player **drags** to steer. Pointer position is normalized to roughly **[-1, 1]** on X and Y (`setPointerFromEvent`). **`updatePlane()`** maps that to:

- **Drag left / right** (**horizontal**): forward **speed** and camera **field of view** (narrower FOV feels like zooming in).
- **Drag up / down** (**vertical**): **flight height** and slight roll/pitch on the plane mesh.

Events are bound on **`document`** (`pointermove`, `pointerup`, `pointercancel`) so both mouse and touch use one code path. The HUD layer uses CSS **`pointer-events: none`** so input passes through to the canvas.

### 3D scene

- **Sea**: A large cylinder, rotated flat, with **per-vertex wave motion** (legacy Three `Geometry` / `.vertices` API — tied to the vendored Three version).
- **Sky**: A ring of **cloud** groups (random cubes), rotated slowly with world speed.
- **Airplane**: Built from boxes with tweaked vertices; **propeller** spins every frame.
- **Coins / enemies**: Placed on a **circular path** using angle + radius; each frame advances the angle and tests distance to the plane. Objects are **pooled** (`ennemiesPool`, `particlesPool`, coin pools inside `CoinsHolder`) to reduce allocation.

### HUD

Glass-style **level** and **distance** cards sit under the title; the **level ring** sits beside the level numeral. **Energy** is a dark “floating” bar **pinned in screen space to the plane** (world position projected through the camera each frame). DOM ids: `#distValue`, `#levelValue`, `#levelCircleStroke`, `#energyBar`, `#planeEnergyHud`. Energy drains with speed; low energy triggers a CSS blink on the fill.

### GitHub Pages

`_config.yml` selects a Jekyll theme; if you publish **only** the static files for this game, you typically deploy the folder contents as static assets (or use a workflow that uploads the site root). If the repo root is built as a Jekyll site, ensure asset paths still resolve for `css/`, `scripts/`, and `fonts/`.

---

## UX features

- **First-run overlay** explains controls; dismissed with **Start game** (stored in `localStorage` as `bottomgun_onboarding_done`).
- **Steer hint** fades in after starting, then hides after a few seconds.
- **Pause** (`Esc` or the Pause control) freezes the scene; **Resume** continues.
- **Web Audio** beeps for coin, hit, level-up, and game over — **off by default**; use **Enable sound** on the intro panel or **Sound off/on** in the corner.
- **HUD**: comma-separated distance, **Low** badge under energy when below 30%, cyan/red **flash** on pickup/hit, level number **pops** on level-up.
- **`prefers-reduced-motion`**: softer HUD flashes, no energy-bar blink, particle bursts skip GSAP.
- **Safe-area** padding and **`touch-action: none`** on the game shell for mobile notches and less accidental scroll.

## Controls

| Input | Effect |
|--------|--------|
| **Drag up / down** | Flight height |
| **Drag left / right** | Forward speed and camera zoom (FOV) |
| **Esc** | Pause / resume (while playing or paused) |
| **Space / Enter** | Restart after game over (when replay is offered) |
| **Tap / click** | Restart after crash (anywhere, or the large replay control) |

---

## Fonts

**Playfair Display** is bundled under `fonts/` (WOFF2) and declared in `css/styles.css`. The typeface is licensed under the [SIL Open Font License](https://scripts.sil.org/OFL) (see the [Playfair Display](https://fonts.google.com/specimen/Playfair+Display) project).
