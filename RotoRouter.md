# RotoRouter — RS/RE Rotation Confirm Fix

## What was wrong
- When using **RS/RE** to rotate an existing *Straight/Elbow* tile:
  - After pressing **Q/E** to adjust rotation and clicking the same cell to confirm, the click was rejected as "Cell occupied".
  - The final rotation was never committed to the board.

## What changed
- **Confirm-before-occupancy:** The click handler now commits RS/RE rotation **before** the `Cell occupied` check.
- **Apply rotation:** On confirm, the tile's `cell.rot` is set to the current `previewRot` (normalized to 0..359).
- **QoL:** When entering rotate mode, the cursor hover is pinned to the rotate target, making the ghost/preview easier to see.

## How to use
1. Draw an **RS** or **RE** card.
2. Click an existing Straight/Elbow tile (matching base type) that is in your *placement highlight*.
3. Press **Q/E** to rotate preview.
4. Click the **same cell** to commit rotation.
5. Or click any empty highlighted cell to place a new tile instead.

## Files
- `RotoRouter_Fixed_RSRE.html` — drop-in replacement.
