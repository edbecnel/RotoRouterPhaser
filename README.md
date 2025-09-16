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


### UI: Save/Load Button Placement — 2025-09-16
- Moved **Save** and **Load** buttons to sit **next to End Turn** in the Actions panel.
- Removed the fallback DOM-insertion that placed them under **Corners**.


### Save Bridge Fix — 2025-09-16
- Guaranteed `window.state` exposure by wrapping `newGame()` and `startTurn()`, so Save can always see `state.board`.
- Snapshot now tries to bridge before erroring.


### Save: Final State Bridge — 2025-09-16
- Exported the internal `state` to `window.state` **at the point of declaration**, so Save always sees `state.board`.
- Kept Save/Load next to **End Turn**.


### Save/Load System Overhaul — 2025-09-16
- Replaced prior broken/duplicate Save/Load implementations with a single **clean module**.
- **Save** and **Load** buttons are now permanently placed next to **End Turn** in the Actions panel.
- Game state (board, players, tokens, decks, scores, turn, phase) is saved to **localStorage** under key `RotoRouterSaveV1`.
- **Save**: Serializes current `state.board`, `state.players`, and transient flags into JSON and stores locally.
- **Load**: Restores from localStorage, rebuilds the board and players, mirrors `reached` sets to UI, clears transient state, and immediately calls `resize()`, `render()`, and `updateHUD()` to refresh the board.
- Status line messages confirm Save/Load actions (e.g., “Game saved to this device.” / “Loaded from this device.”).
- Removed malformed `catch` block and duplicate Save/Load code fragments that caused syntax errors.
