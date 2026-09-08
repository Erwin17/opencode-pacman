# AGENTS.md

Pac-Man clone in vanilla JS + HTML + CSS (no framework, no build tooling). Learning project for spec-driven development.

## Run / verify

- No package.json, no build, no tests, no linter. Open `src/index.html` in a browser to run the game.
- Verify changes by loading the page and playing: arrows move Pac-Man, eat all dots to win.
- There is no automated verification. Grep for console errors manually if needed.

## Architecture

- All files use plain globals, NOT modules/imports. `index.html` loads scripts in order via `<script>` tags — this order is required since later files depend on earlier globals:
  1. `js/maze.js` — exposes `MAZE`, `TUNNEL_ROW`, `PACMAN_START`, `GHOST_STARTS`
  2. `js/game.js` — exposes `createGame`, `update`, `DIRS`; reads maze globals
  3. `js/render.js` — exposes `draw`; reads `DIRS` from game.js
  4. `js/main.js` — game loop, keyboard, overlays; starts on load
- If you add a JS file, it must be added to the `<script>` tags in `index.html` in dependency order.
- `MAZE` is a frozen-at-load grid; each game copies it (`createGame`) into `game.grid` so dots can be eaten without corrupting the original — never mutate `MAZE` directly.
- Grid values: `1`=wall, `2`=dot, `3`=door (blocks Pac-Man but not ghosts), `0`=empty walkable. Maze is 28x31, cell `(x,y)` origin top-left.
- Rendering is cell-aligned: speeds are fractions of a cell per frame (`PACMAN_SPEED = 1/8`, `GHOST_SPEED = 1/10`) so actors re-align to cell centers.

## Conventions

- Spanish is used in user-facing text (overlays, HUD) AND most comments. Keep both in Spanish.
- All variables/functions are global (attached via `window`) and files begin with a `// filename.js` comment stating the file's role and its global dependencies — follow this pattern.
- Style quirk: the codebase puts a space after function names before the paren (`function parseTile( ch )`) and inside parens for calls — match it.
- Working style: spec-driven development. The `spec` and `spec-impl` skills under `.agents/skills/` (locked in `skills-lock.json`) drive the workflow: write an approved spec before code, implement from the spec on a feature branch. Check for an existing spec before changing gameplay behavior.