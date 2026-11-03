# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Vanilla JavaScript Tetris on HTML5 Canvas: `index.html` + `style.css` + `game.js`. No dependencies, no `package.json`, no build step, no test suite, no linter. User-facing text (UI labels, overlay messages, README) is in Spanish.

## Running

Open `index.html` directly, or serve the directory statically (recommended):

```bash
python3 -m http.server 8000   # then http://localhost:8000
```

There are no automated tests; verify changes by playing in the browser.

## Architecture (`game.js`)

- **Single script, global mutable state.** All game state (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropInterval`, `dropAccum`, `animId`) lives in module-level `let`s, reset by `init()`. `init()` is also the restart-button handler.
- **Color index = piece type.** `PIECES[t]` shapes store the value `t` in their filled cells, and `merge()` copies those values into `board`. So `board` cells, shape cells, and `COLORS` indices all share the same 1–7 numbering; index 0 is empty/null in all three arrays. Adding a piece type means extending `PIECES` and `COLORS` together and updating the `* 7` in `randomPiece()`.
- **Collision is the single source of truth.** Movement, rotation (`tryRotate` with kicks `[0, -1, 1, -2, 2]`), soft/hard drop, ghost projection (`ghostY`), and spawn/game-over detection all go through `collide(shape, x, y)`. Cells with `y < 0` are allowed (above the board).
- **Piece lifecycle:** `lockPiece()` → `merge()` → `clearLines()` (updates score/lines/level/`dropInterval`) → `spawn()` (promotes `next` to `current`; if it collides immediately, `endGame()`).
- **Game loop:** `loop(ts)` via `requestAnimationFrame` accumulates elapsed time into `dropAccum` and applies gravity when it exceeds `dropInterval`, then redraws everything. Pause/game-over stop it with `cancelAnimationFrame(animId)`; unpausing resets `lastTime` and calls `loop` directly.
- **Overlay is shared.** The same `#overlay` element shows both PAUSA and GAME OVER; `overlay-title`/`overlay-score` text is swapped per state.

## Coupled values

- Board canvas size in `index.html` (`300×600`) must equal `COLS × BLOCK` by `ROWS × BLOCK`.
- `#next-canvas` (`120×120`) assumes a 4×4 grid at the hard-coded `NB = 30` in `drawNext()`.
- Controls are listed both in the `keydown` handler in `game.js` and in the CONTROLS panel in `index.html` (and the README table) — keep them in sync.
