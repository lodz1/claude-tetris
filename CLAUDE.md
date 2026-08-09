# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Classic Tetris implemented in vanilla JavaScript, HTML5 Canvas, and CSS. No dependencies, no build step, no package.json.

## Running the game

No install/build required — just serve the static files.

```bash
open index.html        # macOS
xdg-open index.html    # Linux
start index.html       # Windows

# or a local static server:
python3 -m http.server 8000
npx serve .
php -S localhost:8000
```

There is no test suite, linter, or build/watch command in this repo.

## Architecture

Three files, no modules/bundler — `game.js` is loaded directly via `<script>` in `index.html` and runs as one top-level script with global state.

- **`index.html`** — DOM structure: `<canvas id="board">` (300×600, the play field), `<canvas id="next-canvas">` (next-piece preview), HUD spans (`#score`, `#lines`, `#level`), and a `#overlay` div reused for both PAUSE and GAME OVER states.
- **`style.css`** — dark/retro arcade visual theme.
- **`game.js`** — all game logic, organized around this flow:
  - `init()` creates the board matrix, seeds `next`, calls `spawn()`, and starts `requestAnimationFrame(loop)`.
  - `loop(ts)` accumulates elapsed time and drops the current piece one row once `dropAccum >= dropInterval`, then calls `draw()` and reschedules itself.
  - Piece movement/rotation and drop actions are wired via the top-level `keydown` listener at the bottom of the file, which dispatches to `tryRotate`, `softDrop`, `hardDrop`, or direct `current.x`/`current.y` mutation, then calls `updateHUD()`.
  - `lockPiece()` → `merge()` (bakes piece into `board`) → `clearLines()` → `spawn()` (promotes `next` to `current`, generates a new `next`; if the new piece immediately collides, calls `endGame()`).

Key data model / algorithms, if modifying game behavior:

- **Board**: `ROWS × COLS` matrix, each cell is `0` (empty) or a color index `1–7` identifying the piece type that occupies it.
- **Pieces**: `PIECES` holds each of the 7 tetrominoes as a square matrix; `COLORS` maps piece index → fill color.
- **Rotation**: `rotateCW` does a transpose + row-reverse. `tryRotate` applies it, then attempts wall kicks at offsets `[0, -1, 1, -2, 2]`, keeping the first offset that doesn't collide.
- **Collision**: `collide(shape, ox, oy)` checks bounds and overlap against `board`; used for movement, rotation, and ghost-piece projection.
- **Ghost piece**: `ghostY()` projects `current` straight down until it would collide; drawn at `globalAlpha = 0.2` in `draw()`.
- **Scoring**: `LINE_SCORES = [0, 100, 300, 500, 800]` indexed by lines cleared at once, multiplied by `level`; hard drop adds 2 pts/row dropped, soft drop adds 1 pt/row.
- **Leveling/speed**: level = `floor(lines / 10) + 1`; `dropInterval = max(100, 1000 - (level - 1) * 90)` ms.

If changing `COLS`, `ROWS`, or `BLOCK` in `game.js`, also update the `width`/`height` attributes of `<canvas id="board">` in `index.html` to match (`COLS × BLOCK` by `ROWS × BLOCK`).

README.md (in Spanish) has additional detail and a controls table.
