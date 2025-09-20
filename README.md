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
- Skip counts are incremented when:
  - A drawn track card is bottomed.
  - A turn ends while still holding a track card.
- Skip counts are reset when:
  - A track card is successfully placed, replaced, or rotated/confirmed (RS/RE).
