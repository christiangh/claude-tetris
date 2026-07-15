# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Vanilla JS Tetris. HTML5 Canvas + CSS. No dependencies, no build step, no tests, no package.json.

## Run

Open `index.html` directly, or serve statically:

```bash
python3 -m http.server 8000   # then open http://localhost:8000
```

No lint/build/test tooling exists.

## Architecture

Three files, all cooperating through DOM element IDs:

- `index.html` — DOM structure. Two canvases: `#board` (300×600) and `#next-canvas` (120×120). HUD spans (`#score`, `#lines`, `#level`) and `#overlay` (pause / game over). `game.js` grabs every one by ID at load — renaming an ID breaks the game silently.
- `style.css` — dark/retro-arcade theme. Purely presentational.
- `game.js` — all game logic (~300 lines, single global scope, `'use strict'`).

### game.js model

- **Board**: `ROWS × COLS` matrix. Each cell is `0` (empty) or a color index `1–7` matching both `COLORS` and `PIECES` array positions. These two arrays are index-aligned — cell value = piece type = color = array index. Keep them in sync.
- **Pieces**: square matrices in `PIECES`. Rotation is transpose+reverse (`rotateCW`); `tryRotate` applies wall kicks by testing offsets `[0,-1,1,-2,2]`.
- **State**: module-level `let` vars (`board, current, next, score, ...`), all reset in `init()`.
- **Loop**: `requestAnimationFrame(loop)` accumulates `dt` into `dropAccum`, drops one row when `dropAccum >= dropInterval`. `lockPiece` → `merge` + `clearLines` + `spawn`. Game over fires when a freshly spawned piece already collides.
- **Speed curve**: `dropInterval = max(100, 1000 - (level-1)*90)`, level = `floor(lines/10)+1`.

## Coupling constraints

Board pixel size is derived, not stored together — three places must agree:

- `COLS`, `ROWS`, `BLOCK` in `game.js`
- `<canvas id="board">` `width`/`height` in `index.html` must equal `COLS*BLOCK` × `ROWS*BLOCK`

Changing `COLS`/`ROWS`/`BLOCK` without updating the canvas attributes clips or misaligns rendering.
