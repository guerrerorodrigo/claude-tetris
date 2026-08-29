# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Classic Tetris implemented in vanilla JavaScript with HTML5 Canvas and CSS. No dependencies, no build step, no `package.json`, no test suite.

## Running

Open `index.html` directly, or serve statically:

```bash
python3 -m http.server 8000
```

Then visit `http://localhost:8000`. There is no build/lint/test command — changes to `game.js`, `index.html`, or `style.css` are picked up on page reload.

## Architecture

Three files, no modules/bundler — `index.html` loads `game.js` as a single classic script that runs immediately (`init()` at the bottom of the file).

- **`index.html`** — DOM shell: `#board` canvas (300×600, 10×20 cells), `#next-canvas` for the next-piece preview, HUD spans (`#score`, `#lines`, `#level`), and the `#overlay` div reused for both PAUSE and GAME OVER states.
- **`style.css`** — dark/retro arcade visual theme.
- **`game.js`** — all game logic, organized around a small set of global `let` bindings (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropInterval`, etc.) mutated by top-level functions rather than a class/state object.

### Core model

- The board is a `ROWS × COLS` matrix; each cell is `0` (empty) or a color index `1–7` identifying which piece type placed it.
- Pieces (`PIECES`) are defined as square matrices; `rotateCW` rotates via transpose + row reversal.
- `collide(shape, ox, oy)` is the single collision check used for movement, rotation, and ghost-piece projection — any change to movement rules should go through it rather than duplicating bounds/overlap logic.
- `tryRotate()` implements basic wall kicks: after rotating, it tries offsets `[0, -1, 1, -2, 2]` and keeps the first that doesn't collide.

### Game loop

`loop(ts)`, driven by `requestAnimationFrame`, accumulates elapsed time in `dropAccum` and advances the piece one row (or calls `lockPiece()`) once `dropAccum >= dropInterval`. `lockPiece()` merges the piece into `board`, clears completed lines, and spawns the next piece via `spawn()`. If the freshly spawned piece immediately collides, `endGame()` fires.

### Scoring/leveling

- Line-clear points come from `LINE_SCORES = [0, 100, 300, 500, 800]`, multiplied by `level`.
- Hard drop adds 2 points per row dropped; soft drop adds 1 point per row.
- `level` increases every 10 cleared lines; `dropInterval` scales down as `max(100, 1000 - (level - 1) * 90)` ms.

### Rendering

`draw()` redraws the whole board each frame: grid lines, locked blocks, the ghost piece (projected via `ghostY()`, drawn at `globalAlpha = 0.2`), then the active piece on top. `drawNext()` renders the preview canvas the same way at a smaller block size.

### Tunable constants (top of `game.js`)

`COLS`, `ROWS`, `BLOCK`, `COLORS`, `LINE_SCORES`, initial `dropInterval` (set in `init()`). If `COLS`/`ROWS`/`BLOCK` change, update the `#board` canvas `width`/`height` in `index.html` to match (`COLS × BLOCK`, `ROWS × BLOCK`).
