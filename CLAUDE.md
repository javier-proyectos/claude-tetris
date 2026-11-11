# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Vanilla JS Tetris (HTML5 Canvas). No `package.json`, bundler, linter, or tests — there is no build/lint/test step. UI text and README are in Spanish.

Run it by opening `index.html` directly, or serve statically: `python3 -m http.server 8000` and visit `http://localhost:8000`.

## Architecture

Three files: `index.html` (DOM + canvases), `style.css`, and `game.js` (all logic, loaded as a classic script, `'use strict'`, global state — no modules).

Key points in `game.js` that span multiple functions:

- **Global mutable state** (`board, current, next, score, lines, level, paused, gameOver, lastTime, dropAccum, dropInterval, animId`) is declared on one line and reset in `init()`. Restart (`#restart-btn`) just calls `init()`.
- **Board/pieces encoding**: `board` is a `ROWS×COLS` matrix of `0` or a color/type index 1–8. `PIECES` and `COLORS` are indexed by the same 1–8 type id (index 0 is `null`), so a piece's cells hold its own type id.
- **Game flow**: `loop` (rAF) accumulates `dropAccum` and gravity-drops or calls `lockPiece()` → `merge()` → `clearLines()` → `spawn()`. `spawn()` promotes `next` to `current` and calls `endGame()` if the new piece collides immediately. Hard/soft drop and keyboard handlers also route through `lockPiece()`.
- **Pause/game over** both stop the loop via `cancelAnimationFrame(animId)` and reuse the single `#overlay` element; `togglePause` restarts `loop` manually.
- **Rotation** (`tryRotate`) is clockwise only, with simple horizontal kicks `[0,-1,1,-2,2]` (not SRS).
- **Scoring/speed** live in `clearLines()` (`LINE_SCORES × level`, level = `lines/10 + 1`, `dropInterval = max(100, 1000 − (level−1)·90)`); drop points are added in `hardDrop` (2/cell) and `softDrop` (1/row).
- **Canvas size coupling**: `#board` in `index.html` is hard-coded to `300×600` (= `COLS×BLOCK` × `ROWS×BLOCK`) and `#next-canvas` to `120×120` (4 cells × 30px in `drawNext`). Changing `COLS`, `ROWS`, or `BLOCK` requires updating these attributes.
