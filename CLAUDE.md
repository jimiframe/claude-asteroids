# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A clone of the classic arcade game **Asteroids**, built with raw HTML5 Canvas and vanilla ES6+ JavaScript. No frameworks, no bundler, no dependencies, no build step, no test suite — everything lives in a single `game.js` file loaded by `index.html`.

## Running it

There is no build/lint/test tooling. To run the game:

- Open `index.html` directly in a browser, or
- Serve it locally: `npx serve .` then visit `http://localhost:3000`

To verify changes, open the page in a browser and play the game (use `/run` or the `verify` skill if available rather than assuming code correctness from reading alone).

## Architecture

Everything is in `game.js`, organized top-to-bottom as:

1. **Input** — `keys`/`justPressed` maps populated by `keydown`/`keyup` listeners; `pressed(code)` consumes a one-shot "just pressed" event (used for shooting and restarting so holding the key doesn't repeat-trigger).
2. **Utils** — `wrap` (toroidal coordinate wrapping), `dist`, `rand`, `randInt`.
3. **Entity classes** — `Bullet`, `Asteroid`, `Ship`, `Particle`. Each has its own `update(dt)` and `draw()`. All entities mark themselves `dead = true` instead of removing themselves; arrays are swept with `.filter(x => !x.dead)` each frame.
4. **Game state** — module-level globals (`ship`, `bullets`, `asteroids`, `particles`, `score`, `lives`, `level`, `state`, `deadTimer`) reset/reassigned by `initGame()` and `nextLevel()`. `state` is a simple FSM: `'playing' | 'dead' | 'gameover'`.
5. **`update(dt)`** — branches on `state` first (gameover/dead have their own minimal update paths), then runs the normal frame: input → entity updates → collision detection (bullet↔asteroid, ship↔asteroid) → level-completion check (`asteroids.length === 0` → `nextLevel()`).
6. **`draw()`** — clears canvas, draws entities back-to-front (particles → asteroids → bullets → ship), then HUD/overlay.
7. **Main loop** — `requestAnimationFrame`-driven `loop(ts)` computing `dt` in seconds (clamped to 0.05 to avoid spiral-of-death on tab-switch), calling `update(dt)` then `draw()`.

### Conventions worth preserving

- **Toroidal space**: all positions wrap around canvas edges via `wrap(v, max)` — `W = 800`, `H = 600` are the fixed canvas dimensions (also set in `index.html`'s `<canvas>` attributes).
- **Size-indexed lookup tables**: `RADII`, `SPEEDS`, `POINTS` are arrays indexed by asteroid `size` (1 = small, 2 = medium, 3 = large; index 0 is unused padding). Splitting an asteroid (`Asteroid.split()`) produces two of `size - 1`, recursing down to nothing at size 1.
- **Dead-flag + filter pattern**: never splice mid-iteration; set `.dead = true` on collision/expiry and filter afterward.
- **`dt`-based motion**: all movement/timers are scaled by `dt` (seconds) for frame-rate independence, not per-frame constants.
- Comments and on-screen text (HUD, overlays) are in **Spanish**, matching the README; keep new user-facing strings consistent with that.
- Keep the project dependency-free and in a single `game.js` unless there's a strong reason to split it up.
