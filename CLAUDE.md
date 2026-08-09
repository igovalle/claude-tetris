# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Vanilla JavaScript Tetris using HTML5 Canvas. No dependencies, no build process, no package.json. The entire game logic lives in a single file: `game.js`.

## Running the game

Open `index.html` directly in a browser, or serve it statically:

```bash
python3 -m http.server 8000
# or
npx serve .
```

There is no build, lint, or test tooling in this repo — changes to `game.js`, `index.html`, or `style.css` take effect on page reload with no compilation step.

## Architecture

Three files cooperate directly via DOM IDs (no modules, no bundler):

- **`index.html`** — DOM structure: `#board` canvas (300×600, i.e. `COLS×BLOCK` by `ROWS×BLOCK`), `#next-canvas` for the piece preview, HUD spans (`#score`, `#lines`, `#level`), and the `#overlay` used for both pause and game-over states.
- **`style.css`** — dark/retro arcade visual theme.
- **`game.js`** — all game state and logic, structured around a handful of core mechanics:
  - **Board model**: `ROWS × COLS` matrix where each cell is `0` (empty) or a color index `1–7` identifying the locked piece type.
  - **Pieces**: defined as square matrices in `PIECES`. Rotation (`rotateCW`) is a transpose + row-reverse, not a lookup table.
  - **Collision** (`collide`): checks board bounds and overlap with locked cells.
  - **Wall kicks** (`tryRotate`): after rotating, tries offsets `[0, -1, 1, -2, 2]` columns until a non-colliding position is found.
  - **Game loop** (`loop`): driven by `requestAnimationFrame`, accumulates elapsed time and drops the piece one row when `dropAccum` exceeds `dropInterval`.
  - **Line clearing** (`clearLines`): scans bottom-up, splices full rows out and unshifts empty rows at the top.
  - **Scoring**: `LINE_SCORES = [0, 100, 300, 500, 800]` multiplied by current level; hard drop awards 2 pts/row, soft drop 1 pt/row.
  - **Leveling/speed**: level increases every 10 lines; `dropInterval = max(100, 1000 - (level-1)*90)` ms.
  - **Ghost piece** (`ghostY`): projects the current piece straight down to its landing row, drawn at `globalAlpha = 0.2`.

Game state (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, timing accumulators) is held in module-level `let` bindings reset by `init()` — there is no state container or class.

Control flow: `init()` → `spawn()` sets `current`/`next` and calls `endGame()` if the new piece immediately collides → `requestAnimationFrame(loop)` drives falling/locking/redrawing every frame → `keydown` handles movement, rotation, soft/hard drop, and pause (`KeyP`) → `restartBtn` click re-runs `init()`.

## Tunable constants (top of `game.js`)

`COLS`, `ROWS`, `BLOCK`, `COLORS`, `LINE_SCORES`, `dropInterval`. If `COLS`/`ROWS`/`BLOCK` change, the `#board` canvas `width`/`height` in `index.html` must be updated to match (`COLS×BLOCK`, `ROWS×BLOCK`).
