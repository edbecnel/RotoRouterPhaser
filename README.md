# RotoRouter — Baseline Package (from user-attached file)

**Built:** 2025-09-16

This package preserves your attached baseline exactly and adds a separate commented edition.

## Files
- `index.html` — **verbatim** copy of your uploaded baseline (no changes)
- `index_commented.html` — same code with a comment header describing:
  - Data model (board cells, players, per-turn flags)
  - Deck composition & placement rules (token-adjacent + owned-adjacent; first placement corner-only)
  - Token movement BFS with *reciprocal openings*
  - Rotation system with *checkerboard meshed-gear parity* and hint overlay
  - Auto-advance and UI locks
- `README.md` — this file

## How to Run
Open `index.html` or `index_commented.html` directly in a modern browser (offline-capable).

## Notes
- All UI from your baseline is retained: setup controls (board size/templates), actions, hover ghost with Q/E rotation,
  connection-edges toggle, toast/status HUD, corner highlights, pulse feedback, and rotation hints.
- The "Block" tile type remains available for templates; the per-player decks use only Straight/Elbow/Cross per your baseline.
