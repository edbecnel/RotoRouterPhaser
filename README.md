# RotoRouter — No-Block Edition

**Built:** 2025-09-16

This edition removes all support for **Block** tracks from your baseline.

## What Changed

- Removed **Block** from the Track enum and connectivity/openings.
- Removed any **deck** additions of Block.
- Removed **UI** items that mention Block (buttons/options/etc.).
- Neutralized any stray "Block" references to **Empty/null** via a safety patch.
- Token movement, placement, rotation, edges, and the rest are **unchanged**.

## Files

- `index_noBlock.html` — playable no-Block build
- `index_noBlock_commented.html` — same build with a change log and notes
- (Original baseline remains in your previous package as `index.html`)

## Run

Open either HTML file directly in a browser.

---

## Fixes & Enhancements

### [2025-09-16] RS/RE Rotation Confirm Fix

**What was wrong**

- When using **RS/RE** to rotate an existing _Straight/Elbow_ tile:
  - After pressing **Q/E** to adjust rotation and clicking the same cell to confirm, the click was rejected as "Cell occupied".
  - The final rotation was never committed to the board.

**What changed**

- **Confirm-before-occupancy:** The click handler now commits RS/RE rotation **before** the `Cell occupied` check.
- **Apply rotation:** On confirm, the tile's `cell.rot` is set to the current `previewRot` (normalized to 0..359).
- **QoL:** When entering rotate mode, the cursor hover is pinned to the rotate target, making the ghost/preview easier to see.

**How to use**

1. Draw an **RS** or **RE** card.
2. Click an existing Straight/Elbow tile (matching base type) that is in your _placement highlight_.
3. Press **Q/E** to rotate preview.
4. Click the **same cell** to commit rotation.
5. Or click any empty highlighted cell to place a new tile instead.

---

### [2025-09-20] HUD Skipped Counter + Visual Warning

- Added a new `<span id="skippedTag">` to the HUD, displayed between **Token** and **Score**.
- `updateHUD()` now updates `skippedTag` each turn with the player’s current skip count (`Skipped: x/3`).
- When the skip count reaches **3 or more**, the `warn` CSS class is applied, rendering the Skipped counter in bold red text for clear visual feedback.
- A `.warn` CSS class was added to the stylesheet:
  ```css
  .warn {
    color: #c00;
    font-weight: 600;
  }
  ```
