# AGENTS.md

## What this is
Pac-Man clone in **Vanilla JS + HTML + CSS**. No `package.json`, no build, no deps,
no tests, no linter, no CI, **no git remote** (commits are local only, never push).
`README.md` and all code comments are in **Spanish** (no accents) — keep it that way.

This repo is a spec-driven-development exercise, so work normally starts with a spec,
not with code.

## Run / verify
- Open `src/index.html` in a browser, or `python3 -m http.server -d src 8000` from the root.
- Verification is manual: play it and read the console. `git diff` is the only other check.

## Workflow: spec first
- Specs live in `specs/NN-kebab-slug.md` (the folder does not exist yet), state machine
  `Draft → Approved → Implemented`, one branch per spec: `spec-NN-slug`.
- The `/spec` and `/spec-impl` skills in `.agents/skills/` (versions pinned in
  `skills-lock.json`) own that flow: questions before writing, no code during `/spec`,
  no auto-commits, and `/spec-impl` refuses any spec not in `Approved`.
- `specs/.spec-config.yml` (`AutoCreateBranch`) does not exist yet → default `true`.

## Architecture: plain globals, not modules
`src/index.html` loads scripts in this exact order, as classic `<script>` tags:
`maze.js → game.js → render.js → main.js`. Cross-file deps are `window` globals, so the
order is load-bearing. **Do not convert to ES modules** — that breaks the `file://` workflow.

- `maze.js` — `MAZE_STR`, 31 strings of 28 chars, parsed to numeric `MAZE`. Tile codes:
  `1` wall · `2` dot · `3` ghost pen door · `0` empty walkable. Origin top-left,
  `x∈[0,27]`, `y∈[0,30]`, mirror-symmetric about the vertical center axis.
  Exposes `MAZE`, `TUNNEL_ROW`, `PACMAN_START`, `GHOST_STARTS`.
- `game.js` — rules. `createGame()` copies `MAZE` into `game.grid` and **mutates only that
  copy** (dot eating), so `MAZE` stays pristine and a restart is just `createGame()`.
  Exposes `createGame`, `update`, `DIRS` (`render.js` also needs `DIRS`).
- `render.js` — `draw(ctx, game, frame)`, `TILE = 20`. Reads `game.grid`, never `MAZE`.
- `main.js` — `requestAnimationFrame` loop, arrow keys → `pacman.nextDir`, and the DOM
  overlay for start/win/lose. `showOverlay` rewrites `#overlay`'s innerHTML and re-binds a
  fresh `#action-btn`, so the module-level `actionBtn` ref goes stale after the first overlay.

## Invariants that break the game if ignored
- Movement is continuous in cell units with fractional speeds (`0.125` = 1 cell / 8 frames).
  Turns, dot eating and ghost decisions happen **only when the actor is on a cell center**
  (`aligned()`, 1e-3 epsilon). Skip that check and rounding bugs appear.
- `isWall(grid, x, y, actor)`: ghosts pass the pen door (`3`), Pac-Man does not.
- Tunnel wrap applies only on `TUNNEL_ROW`.
- Canvas attributes (560x620) must equal `cols*20 x rows*20`. Changing maze dimensions
  means editing `index.html` too.
- `game.state` is `'start' | 'playing' | 'won' | 'lost'`; `main.js` is the only consumer.

## Code style actually used here
2-space indent, single quotes, semicolons, `const`/plain functions (no classes).
Spacing is WordPress-ish: `[ x, y ]`, `( a + b )`, `!cond` for negation.
Each file starts with a header comment naming the file and its dependencies.