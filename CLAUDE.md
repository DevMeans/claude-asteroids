# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A clone of the arcade game **Asteroids** built with raw HTML5 Canvas and vanilla ES6+. No framework, no bundler, no dependencies, no build step, and no test suite. All game logic lives in a single file: `game.js` (~420 lines). `index.html` just sizes the 800×600 canvas and loads the script.

## Running

There is nothing to build. Either open `index.html` directly in a browser, or serve the folder:

```bash
npx serve .
```

Then visit the printed URL (typically `http://localhost:3000`). There are no lint or test commands — verification is done by loading the page and playing.

## Architecture

Everything is global module scope inside `game.js`, organized top-to-bottom:

- **Input** — global `keys` (held) and `justPressed` (edge) maps populated by `keydown`/`keyup` listeners. Use `keys['ArrowUp']` for continuous actions and `pressed('Space')` for one-shot actions; `pressed()` consumes the edge so it fires once per keypress.
- **Entity classes** — `Bullet`, `Asteroid`, `Ship`, `Particle`. Each follows the same contract: a constructor, `update(dt)`, `draw()`, and a `dead` boolean. Adding a new entity type means following this same shape and wiring it into the arrays in `update()`/`draw()`.
- **Game state** — module-level `ship, bullets, asteroids, particles` arrays plus `score, lives, level, state, deadTimer`. `state` is one of `'playing' | 'dead' | 'gameover'` and `update()` branches on it.
- **Game loop** — `loop(ts)` uses `requestAnimationFrame` and computes a delta-time `dt` (clamped to 0.05s to avoid tunneling on lag/tab-switch). Calls `update(dt)` then `draw()`. All movement/physics is `dt`-scaled, so tune behavior via the constants, not frame counts.

### Conventions to preserve

- **Coordinates wrap toroidally.** Every entity's `update()` runs position through `wrap(v, max)` so objects leaving one edge reappear on the opposite. New moving entities must do the same.
- **Dead-flag + filter lifecycle.** Entities are never spliced mid-iteration. They set `this.dead = true`, and `update()` rebuilds each array with `.filter(e => !e.dead)`. Asteroid splitting collects children in `newAsteroids` and concats after the collision loop — don't mutate the array being iterated.
- **Per-size lookup tables.** Asteroids use parallel arrays `RADII`, `SPEEDS`, `POINTS` indexed by `size` (1=small, 2=medium, 3=large). Keep all three in sync when changing sizes. Note scoring is inverted: smaller asteroids are worth more.
- **Tunable constants are inline** as `const` inside the relevant method/class (e.g. `THRUST`, `DRAG`, `ROT` in `Ship.update`, `SPEED`/`ttl` in `Bullet`). Gameplay tuning happens there.
- **Rendering is immediate-mode** stroked vector art (mostly `#fff` on `#000`) using `ctx.save()/translate/rotate/restore`. No sprites or images.

## Note on docs

`README.md` (Spanish) still mentions power-ups and a "shooting star" asteroid type — these were removed (commit `13e713f`) and no longer exist in `game.js`. Trust the code over the README for current features.
