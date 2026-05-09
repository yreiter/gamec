# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

**NEON VOID** is a single-file browser space shooter. The entire game lives in `index.html` — HTML structure, CSS, and JavaScript are all in one file. There is no build system, no package manager, and no test suite.

## Running the Game

Open `index.html` directly in a browser (file:// works) or serve it with any static server:

```bash
python3 -m http.server 8000
# then open http://localhost:8000
```

## Architecture

The JavaScript in `index.html` is organized into clearly labeled sections (marked with `═══` banners):

| Section | Responsibility |
|---|---|
| CANVAS | Canvas sizing and responsive resize |
| AUDIO | Web Audio API `tone()` helper and `sfx` sound map |
| SHIPS | `SHIPS` array — 3 ship definitions, each with a `fire()` method |
| DIFFICULTIES | `DIFFS` array — 3 difficulty presets with multiplier fields |
| LEADERBOARD | localStorage persistence under key `nv_lb_v3` |
| TITLE SCREEN UI | DOM construction for ship cards and difficulty buttons |
| GAME STATE | Global mutable state: `score`, `wave`, `lives`, entity arrays |
| INPUT | Keyboard (`keys` map), mouse, and touch handlers |
| HUD | `updateHUD()` syncs DOM elements to game state |
| GAME FLOW | `startGame()`, `gameOver()`, `takeDamage()`, screen transitions |
| UPDATE | `update(dt)` — physics, collision detection, wave progression |
| RENDER | `render()` — full Canvas 2D draw pass |
| LOOPS | `loop()` (gameplay) and `idleLoop()` (title screen animation) |

## Key Conventions

**Delta time**: All movement and timers are multiplied by `dt`, which is normalized to 60 fps (`(elapsed_ms / 16.67)`), clamped to 3. Enemy fire rates and cooldowns are expressed in ticks at 60 fps.

**Entity arrays**: `bullets`, `enemyBullets`, `enemies`, `powerups`, `particles`, `stars` are module-level arrays. They are cleared on `startGame()`. Iteration always goes reverse (`for let i = arr.length-1; i>=0; i--`) when splicing during the loop.

**Collision detection**: All collisions use squared distance (`dx*dx+dy*dy < r*r`) — no `Math.sqrt` in the hot path. Player hit radius is `player.w * activeShip.hitR` (smaller than the visual ship, ship-specific).

**Leaderboard key**: The localStorage key is versioned (`nv_lb_v3`). If the score schema changes, increment this key to avoid deserializing stale data.

**Wave progression**: Every 5th wave (`w % 5 === 0`) spawns a single boss instead of a grid. Boss HP scales with `(base + (wave-5)*6) * difficulty.eHp`. Non-boss waves scale grid size by wave number.

**Adding a new ship**: Add an entry to `SHIPS` with `id`, `name`, `tagline`, `color`, `speed`, `baseCooldown`, `hitR`, `stats` (3-element array for SPD/FIRE/PWR bar widths), and a `fire(player, bullets)` method. The title-screen card and in-game rendering both branch on `activeShip.id`.

**Adding a new power-up**: Add an entry to `PU_TYPES` with `id`, `color`, `label`, `icon`, then add the activation logic to `applyPowerup()`. Duration is stored as a frame-count on the `player` object; decrement it in the `update()` player section.

**Audio**: `getAC()` lazily creates the `AudioContext` on first user gesture. All sound is synthesized via `tone(freq, type, dur, vol, atk, dcy)` — no audio files.
