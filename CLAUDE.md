# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the game

No build step. Open `index.html` directly or serve with any static server:

```bash
python3 -m http.server 8000
# then open http://localhost:8000
```

## Architecture

Three files, no dependencies, no bundler:

- `index.html` — DOM structure: `<canvas id="board">` (300×600px) for the game, `<canvas id="next-canvas">` (120×120px) for the next-piece preview, a sidebar panel with score/lines/level, and a shared overlay div for both PAUSE and GAME OVER states.
- `style.css` — Dark/retro theme with flexbox layout.
- `game.js` — All game logic (~300 lines, `'use strict'`).

### game.js key concepts

**State**: a flat set of `let` variables (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropInterval`, `dropAccum`, `lastTime`, `animId`). `init()` resets all of them.

**Board**: `ROWS×COLS` matrix; cells hold `0` (empty) or a color index `1–7` matching `COLORS`/`PIECES` arrays.

**Pieces**: defined as 2D matrices in `PIECES[1..7]`. Clockwise rotation via `rotateCW` (transpose + reverse rows). `tryRotate` attempts kicks `[0, -1, 1, -2, 2]` before giving up.

**Game loop**: `requestAnimationFrame`-based. `loop(ts)` accumulates delta time in `dropAccum`; when it exceeds `dropInterval` the piece drops one row or locks. Speed: `max(100, 1000 − (level−1)×90)` ms.

**Piece lifecycle**: `spawn()` → gravity/input → `lockPiece()` → `merge()` + `clearLines()` → `spawn()`. If `spawn` immediately collides → `endGame()`.

**Rendering**: `draw()` clears canvas, draws grid, locked board blocks, ghost piece at `ghostY()` with `globalAlpha=0.2`, then the active piece. `drawBlock` uses `COLORS[colorIndex]` plus a white highlight strip.

### Tunable constants (top of game.js)

| Constant | Default | Note |
|---|---|---|
| `COLS` / `ROWS` | 10 / 20 | Must match canvas `width`/`height` in HTML (`COLS×BLOCK` / `ROWS×BLOCK`) |
| `BLOCK` | 30 | Pixel size per cell |
| `COLORS` | 7 colors | Index-matched to `PIECES` |
| `LINE_SCORES` | `[0,100,300,500,800]` | Multiplied by current level |
