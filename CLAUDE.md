# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Vanilla Tetris implementation using HTML5 Canvas. No dependencies, no build system, no package manager, no tests — just `index.html`, `style.css`, and `game.js`.

## Running

Open `index.html` directly in a browser, or serve it locally (`python3 -m http.server 8000`, `npx serve .`, etc.) and visit the served URL. There is no build, lint, or test step to run.

## Architecture

Everything lives in `game.js` (~300 lines), driven by a single `requestAnimationFrame` loop (`loop()`). Key pieces, in the order data flows through them:

- **Board model**: `board` is a `ROWS × COLS` matrix; each cell is `0` (empty) or a color index 1–7 identifying which piece locked there. `COLORS[index]` maps that back to a hex color.
- **Pieces**: `PIECES` defines the 7 tetrominoes as square matrices (color-index values, not booleans). `randomPiece()` clones a shape into a `{ type, shape, x, y }` piece object. Rotation (`rotateCW`) is a transpose + row-reverse of the shape matrix.
- **Collision** (`collide(shape, ox, oy)`): the single source of truth for "can a shape occupy this position" — used for movement, rotation, ghost-piece projection, and drop detection. Any new movement logic should go through this rather than reimplementing bounds/overlap checks.
- **Wall kicks** (`tryRotate`): after rotating, tries offsets `[0, -1, 1, -2, 2]` against `collide` and keeps the first that fits.
- **Locking** (`lockPiece` → `merge` + `clearLines` + `spawn`): merges the current piece into `board`, clears completed rows (shifting everything down, unshifting empty rows at top), then spawns the next piece. `spawn()` promotes `next` to `current`, generates a new `next`, and calls `endGame()` if the new piece immediately collides.
- **Scoring/level**: `LINE_SCORES` (`[0,100,300,500,800]`) times `level` on line clears; hard drop adds 2 pts/row dropped, soft drop 1 pt/row. `level` increments every 10 lines and drives `dropInterval = max(100, 1000 - (level-1)*90)`.
- **Rendering** (`draw`, `drawNext`, `drawGrid`, `drawBlock`): board, ghost piece (`ghostY()` projects straight down via `collide`, drawn at `alpha 0.2`), and the falling piece are redrawn every frame on `<canvas id="board">`; the next-piece preview draws separately on `<canvas id="next-canvas">`.
- **Input**: a single `keydown` listener switches on `e.code` for movement/rotate/soft-drop/hard-drop; `P` toggles pause independent of `gameOver`/`paused` state via `togglePause()`.

`index.html` only holds structure (board canvas, next-piece canvas, HUD, controls list, pause/game-over overlay). `style.css` is presentation only (dark/retro theme). All game state and logic stays in `game.js` — keep it that way rather than pulling logic into markup or styles.

Tunable constants live at the top of `game.js`: `COLS`, `ROWS`, `BLOCK`, `COLORS`, `LINE_SCORES`, initial `dropInterval`. If `COLS`, `ROWS`, or `BLOCK` change, update the `<canvas id="board">` `width`/`height` in `index.html` to match (`COLS × BLOCK`, `ROWS × BLOCK`).
