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
