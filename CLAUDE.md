# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository overview

This repo contains a single self-contained HTML5 game: `asteroid-shooter.html`. There is no build system, package manager, bundler, or test suite — HTML, CSS, and JavaScript all live in one file with no external dependencies.

## Development workflow

- **Run/preview**: open `asteroid-shooter.html` directly in a browser (double-click, or `open asteroid-shooter.html` / drag into a browser tab). No server or build step is required.
- **Edit**: modify the file in place. There is no linter, formatter, or test command configured for this repo — verify changes by reloading the file in a browser and playing through the affected behavior (movement, shooting, collisions, touch controls, responsive scaling, etc.).
- There is no `package.json`, CI config, or dependency manifest.

## Code architecture

`asteroid-shooter.html` is organized into clearly delimited sections (marked with `// ── Section ──` comments) within a single `<script>` block:

- **Canvas setup / responsive scaling** — fixed internal resolution (`W=900, H=620`); `resize()` scales the canvas via CSS to fit the viewport while preserving aspect ratio.
- **Input handling** — two parallel input sources feed a shared `keys`/`touchKeys` state, unified by the `key(code)` helper: keyboard listeners (`keydown`/`keyup`) and on-screen touch buttons (`bindBtn`, shown only when `isTouchDevice` is detected). Both write into logical key codes (`ArrowLeft`, `ArrowRight`, `Space` for thrust, `KeyS` for fire).
- **Constants** — tunable gameplay values (rotation speed, thrust, drag, bullet speed/lifetime, shield invincibility duration, etc.) are grouped near the top of the script.
- **Global mutable state** — `state` (a string state machine: `'title' | 'playing' | 'dead' | 'gameover'`), plus `ship`, `bullets`, `asteroids`, `particles`, `debris`, `score`, `shields`, `level`, `asteroidCap`. All game objects are plain arrays of plain objects (no classes).
- **Entity factories** — `spawnAsteroid(size, x, y)` creates asteroids at a given size (`large`/`medium`/`small`, defined in `SIZES` with radius/speed/score) either at explicit coordinates or off-screen edges; `makeShape(r)` generates a randomized polygon outline. `burst()` and `shipDebris()` create particle/debris effects.
- **`initGame()`** resets all state to start a new run.
- **`update()` / `updatePlaying()`** — the per-frame simulation step: reads input, applies thrust/drag/rotation to the ship, wraps positions at screen edges (`wrap()`), handles shooting cooldown, and runs collision detection (bullet↔asteroid splits large→medium→small; ship↔asteroid consumes shields and triggers `state = 'dead'` → `'gameover'` on depletion). Also handles asteroid respawn rate and level progression (`asteroidCap` grows over time).
- **Draw functions** (`drawShip`, `drawAsteroid`, `drawHUD`, `drawOverlay`) — pure rendering based on current state; `drawOverlay` renders the title screen and game-over screen text/prompts, with different copy for touch vs. keyboard players.
- **`loop()`** — the `requestAnimationFrame` game loop: calls `update()`, then draws background/stars/particles/debris/asteroids/bullets/ship/HUD/overlay in that back-to-front order.

When making changes, keep new logic within the existing sectioned structure rather than introducing new files, modules, or build tooling — the project is intentionally a single dependency-free HTML file.
