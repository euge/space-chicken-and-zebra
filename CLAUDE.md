# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Single-file HTML5 space platformer game for kids (target age: 8). Zero dependencies — everything lives in `index.html` (~2100 lines of HTML/CSS/JS). `image.png` is the reference art for the cartoony retrofuturistic style.

## Running the Game

Open `index.html` directly in a browser, or serve locally:
```
python3 -m http.server 8080
```
No build step, no npm, no transpilation.

## Architecture

The entire game is a single `index.html` with three sections:

1. **CSS** (inline `<style>`) — UI overlays, responsive layout, touch controls
2. **HTML** — Screen overlays (title/select/HUD/complete/gameover/win), `<canvas id="game">`, touch buttons
3. **JavaScript** (inline `<script>`) — All game logic:
   - **Constants & state** — `TILE=40`, `GRAVITY=0.55`, character definitions (chicken/zebra), game state machine
   - **Drawing functions** — `drawChicken()`, `drawZebra()`, `drawPlatform()`, `drawEnemy()`, etc. (all canvas 2D)
   - **Level builder** — `buildLevel(n)` constructs platforms/collectibles/enemies/goals for levels 0-4
   - **Physics & collision** — One-way platforms (pass through from below, land on top), AABB collision, coyote time, jump buffering
   - **Game loop** — Fixed 60Hz timestep via `requestAnimationFrame` with delta accumulator
   - **Input** — Keyboard (arrows/WASD/space), touch (multi-touch buttons), pause (P/Escape)
   - **Audio** — Web Audio API synth beeps (jump, collect, hurt, checkpoint, level complete)
   - **Persistence** — `localStorage` for level progress, score, character choice

## Key Design Decisions

- **One-way platforms**: Players jump through platforms from below and land on top. This is critical for vertical level sections.
- **Two characters**: Chicken (double-jump/flutter) vs Zebra (higher single jump, faster). Both must be able to complete all 5 levels.
- **Kid-friendly physics**: Coyote time (7 frames), jump buffering (7 frames), 80% width hitboxes, variable jump height (hold for full, tap for short).
- **Palette**: teal background `#1a4a4a`, orange accent `#e8741e`, cream text `#f5f0e8`, dark outlines `#2d2d2d`.

## Level Structure

Levels are defined procedurally in `buildLevel()` using tile coordinates (multiply by `TILE=40` for pixels):
- `P(x, y, w, h)` — static platform
- `C(x, y, type)` — collectible (star/moon/gem)
- `E(x, y, x1, x2)` — UFO enemy with patrol range
- `MP(x, y, w, h, moveX, moveY, dist)` — moving platform
- `CP(x, y)` — checkpoint
- `G(x, y)` — goal (rocket ship)

When adding/modifying levels, verify all gaps are jumpable for both characters. Chicken max jump distance ~160px (single) or ~288px (with flutter). Zebra max ~220px.
