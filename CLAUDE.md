# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Asteroids clone in plain HTML5 Canvas + vanilla JS. No dependencies, no bundler, no tests, no linter, and not a git repo. The README and in-game text are in Spanish; match that for user-facing strings.

## Running

Open `index.html` directly in a browser, or serve with `npx serve .` (http://localhost:3000). There is no build step.

## Architecture

All game logic lives in `game.js` (loaded by `index.html` onto a fixed 800x600 canvas; `W`/`H` constants must match the canvas size). Structure, top to bottom:

- **Input**: `keys` (held state) and `justPressed` (edge-triggered). Read edge-triggered input via `pressed(code)`, which consumes the flag. Used for Space restart; shooting reads held `keys['Space']` and is rate-limited by `ship.shootCooldown`.
- **Entities** (`Bullet`, `Asteroid`, `PowerUp`, `Ship`, `Particle`): each has `update(dt)`, `draw()`, and a `dead` flag. Dead entities are removed by array `filter` in `update()`, not inside the entity. Positions wrap toroidally via `wrap()` (except particles, which don't wrap).
- **Asteroid sizes** are 1-3 and index the parallel `RADII` / `SPEEDS` / `POINTS` arrays (index 0 unused). `split()` spawns two of `size - 1`.
- **Game state**: module-level globals (`ship`, `bullets`, `asteroids`, `particles`, `powerups`, `score`, `lives`, `level`) plus `state` in `'playing' | 'dead' | 'gameover'`. `'dead'` is a 2s respawn timer during which asteroids and particles keep updating. `initGame()` resets everything; `nextLevel()` spawns `3 + level` asteroids when none remain.
- **Loop**: `requestAnimationFrame` with `dt` in seconds, clamped to 0.05 to avoid big jumps after tab switches. All motion is dt-scaled (except the ship's `DRAG`, applied per frame).
- **Collisions**: circle-based using `dist()`. Ship hit uses `a.radius * 0.82` as a forgiving hitbox and is skipped while `ship.invincible > 0`.

## Notes

- The Shield power-up is implemented (`PowerUp` class, `ship.shield` timer, `destroyAsteroid()` rolls the 10% drop). The README also mentions other power-ups and "estrella fugaz" asteroids; those are not implemented in `game.js`.
