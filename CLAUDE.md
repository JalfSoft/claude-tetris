# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Vanilla JS Tetris on HTML5 Canvas. No dependencies, no `package.json`, no build, no tests, no linter. Three files: `index.html` (DOM + canvases), `style.css`, `game.js` (all logic). UI text and README are in Spanish — keep new user-facing strings in Spanish.

## Running

Open `index.html` directly (`start index.html` on Windows) or serve statically:

```bash
python -m http.server 8000   # then http://localhost:8000
```

Verification is manual in the browser.

## Architecture (`game.js`)

- **Global mutable state**: `board, current, next, score, lines, level, paused, gameOver, lastTime, dropAccum, dropInterval, animId` declared once at top; `init()` resets all of them and is also the restart handler.
- **Board**: `ROWS × COLS` matrix; `0` = empty, `1–8` = piece type, which doubles as index into `COLORS` and as the value stored inside `PIECES` shape matrices. Adding/changing a piece means keeping `PIECES[i]` cells, `COLORS[i]`, in sync (`randomPiece()` derives from `PIECES.length`). Piece 8 is the 3×3 "Tuerca" ring with an empty center, spawned with probability `NUT_CHANCE` (5%); the other 7 are uniform.
- **Pieces**: `{ type, shape, x, y }`; `shape` is a square matrix copied from `PIECES`. Rotation = `rotateCW` (transpose + reverse) then `tryRotate` tests horizontal kicks `[0, -1, 1, -2, 2]` via `collide`. Not SRS.
- **`collide(shape, ox, oy)`** is the single source of truth for movement validity; cells with `ny < 0` are allowed (above board).
- **Lock pipeline**: `lockPiece()` → `merge()` → `clearLines()` (updates lines/score/level/`dropInterval`) → `spawn()` (promotes `next`, detects game over on spawn collision, redraws next-preview).
- **Loop**: `requestAnimationFrame(loop)` accumulates `dt` into `dropAccum`; gravity step when `dropAccum >= dropInterval`. Pause/game over use `cancelAnimationFrame(animId)`; resume calls `loop()` directly after resetting `lastTime`.
- **Rendering**: full redraw each frame in `draw()` (grid, board, ghost at alpha 0.2, current piece); `drawBlock` is shared with the 120×120 `#next-canvas` preview (`drawNext`, 4×4 cells of 30px).
- **HUD/overlay**: `updateHUD()` writes DOM spans; `#overlay` is shown for both PAUSA and GAME OVER by toggling the `hidden` class.

## Constraints

- `#board` canvas size in `index.html` is hard-coded to `COLS*BLOCK × ROWS*BLOCK` (300×600). Changing `COLS`, `ROWS`, or `BLOCK` requires updating the canvas `width`/`height` too.
- Scoring: `LINE_SCORES[cleared] * level`; hard drop +2/cell, soft drop +1/row. Level = `floor(lines/10)+1`; `dropInterval = max(100, 1000 - (level-1)*90)`.
- Controls (`keydown`, by `e.code`): arrows, `KeyX` rotate, `Space` hard drop, `KeyP` pause. The controls list in `index.html` and the README table mirror these — update together.
