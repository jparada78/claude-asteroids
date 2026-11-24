# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

An Asteroids-style arcade game on a plain HTML5 canvas. No dependencies, no bundler, no build step, no tests, no linter. In-game text, comments and the README are in Spanish. Keep new UI strings in Spanish too.

## Running

Open `index.html` directly in a browser, or serve the folder:

```bash
npx serve .   # then visit http://localhost:3000
```

A demo is deployed at https://klerith.github.io/claude-asteroids/ (GitHub Pages).

## Architecture

All game logic lives in `game.js`, a single non-module `'use strict'` script loaded by `index.html`. It uses globals: `canvas`, `ctx`, `W`/`H` (800×600, which must match the `<canvas>` size in `index.html`), and the game state variables.

- **Loop:** `requestAnimationFrame(loop)` calls `update(dt)` and then `draw()` each frame. `dt` is in seconds and capped at 0.05 s, so all speeds are px/s and all timers count down in seconds.
- **State machine:** the global `state` is `'playing'`, `'dead'` (a respawn countdown on `deadTimer`, during which asteroids keep moving) or `'gameover'` (Space restarts through `initGame()`). `update()` branches on `state` first.
- **Entities:** the `Bullet`, `Asteroid`, `Ship` and `Particle` classes each have `update(dt)` and `draw()`, and draw straight onto the global `ctx`. Removal is deferred: an entity sets `this.dead = true`, and `update()` filters the arrays (`bullets`, `asteroids`, `particles`) afterwards. Follow the same pattern for new entity types.
- **World:** the playfield wraps at the edges (toroidal), using `wrap()`. Particles are the exception and do not wrap.
- **Asteroid sizes:** `size` is 3 (large), 2 (medium) or 1 (small), and indexes the lookup arrays `RADII`, `SPEEDS` and `POINTS` (index 0 is unused). `split()` returns two asteroids of `size - 1`.
- **Collision:** circle-distance checks with `dist()`. The ship's hitbox uses `a.radius * 0.82` to make it more forgiving. The ship is invulnerable while `ship.invincible > 0`, and it blinks during that time.
- **Input:** the `keys` object holds keys being held down (used for rotation and thrust). `pressed(code)` is a one-shot edge trigger that consumes `justPressed`; use it for discrete actions such as shooting or restarting.
- **Levels:** when `asteroids` is empty, `nextLevel()` spawns `3 + level` large asteroids, all outside `SAFE_DIST` from the center, where the ship respawns.

## Notes

- The README's description still mentions power-ups and a "shooting star" asteroid. Those were removed in commit `13e713f` and no longer exist in the code.
