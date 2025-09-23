# RotoRouter — No-Block Edition

**Built:** 2025-09-23

This build removes all support for **Block** tracks and adds gameplay/HUD fixes to eliminate soft-locks around skip limits. It also introduces rules to (1) **force the introduction of elbows** after players skip them three times and (2) prevent **corner deadlocks** by allowing opponents to score on a corner even if the owner’s token is sitting there.

---

## Quick start

- Open **`RotoRouter.html`** in any modern desktop browser.
- Recommended: hard-refresh (Ctrl/Cmd+Shift+R) after swapping files so your browser doesn’t cache an older build.

---

## What changed (high level)

- **Block tracks removed** from rules, decks, and UI (non-destructive to the rest of gameplay).
- **Forced Draw at 3/3 track skips**: after three skipped track placements in a row, the player must **Draw + Place** before ending their turn.
- **Elbows Skipped (3/3) forced placement**
  - Tracks how many times a player **skips placing an Elbow** (either by bottoming an **Elbow** or ending their turn while holding an **Elbow**).
  - After **3**, the next **Elbow** drawn **must be placed** that turn (no Bottom, no End Turn, no Override).
- **NEW — Corner non-blocking rule**
  - If a player has a token **on their own corner**, it **does not block** another player from moving onto that corner to score.
  - Owner’s corner token **remains**; the scoring token is removed after scoring (as usual). No deadlocks.
- **HUD & buttons**
  - “Skipped: x/3” with **⚠ Maximum skipped** at 3/3.
  - “Elbows Skipped: x/3” with **⚠ Next Elbow must be placed** at 3/3.
  - **Draw: Ready (Forced)** label when forced-draw and no card in hand.
  - **Place** enabled only when a track card is in hand; **Bottom** disabled when empty/forced; **End Turn** disabled while forced.

---

## Detailed behavior & UX

### Forced Draw (after 3/3 track skips)

When a player’s `skipCount ≥ 3` **and** they have **no drawn card**:

- **Draw** is **enabled** and labeled **“Draw: Ready (Forced)”**.
- **End Turn** is **disabled**.
- **Override** is **hidden** (no card to act on yet).
- **Bottom** is **disabled** (no card; bottoming is irrelevant while forced).
- **Place** is **disabled** (no card to place).

After the player **Draws** while forced:

- **End Turn** remains **disabled** until they **Place**.
- **Override** becomes **visible** (only while a card is in hand).
- **Draw** should **not** flip to “Used” solely by drawing while forced; it stays available until a placement path resolves the force.
- **Place** becomes **enabled** (since a card is now in hand).

When the player successfully **Places** (or uses RS/RE/RC to complete an action that consumes the card), the system clears the forced state:

- `skipCount` resets to **0**.
- `state.forcePlace` becomes **false**.
- Buttons/HUD return to normal for the next turn.

### Elbows Skipped (3/3) forced placement

- Each time a player **skips placing an Elbow** (either by **Bottom** on an **Elbow** card **or** ending the turn while **holding** an **Elbow**), increment **`elbowSkipCount`**.
- When **`elbowSkipCount ≥ 3`**, set a **pending elbow-forced** flag for that player.
- The **next time that player draws an Elbow**, they are **forced to place it** that turn:
  - **End Turn** is **disabled**.
  - **Bottom** is **disabled**.
  - **Place** remains **enabled**; highlights show legal cells as usual.
  - **Override** is **hidden** (no bypass).
- After the Elbow is successfully placed: reset `elbowSkipCount = 0` and clear the elbow-forced flag.

> The elbow-forced flag **persists across turns and non-elbow draws**; it only clears once the player places an Elbow.

### NEW — Corner non-blocking rule (prevents deadlock)

- If a corner currently contains **its owner’s token**, **opponents may still move onto that corner to score**.
- The **owner’s token stays** on the corner (it is not removed).
- The scoring player's token is removed as part of the usual scoring resolution, and their score is updated.

---

## Developer notes (function-level pointers)

> Names below refer to functions/logic in `RotoRouter.html`. Keep these invariants intact.

### `checkForcePlace()` (track skips)
- Sets `state.forcePlace = (P.skipCount ≥ 3)`.
- If forced **and** **no card in hand**: clears `P.drawLocked` and `P.drawUsed` so **Draw** is usable and not marked “Used.”
- Call this **before** rendering the HUD whenever skip count or hand state might change.

### `checkForceElbow(P)` (elbow skips)
- Sets `P.forceElbow = (P.elbowSkipCount ≥ 3)`; persists across turns.
- Does **not** change draw locks—elbow forcing only applies when an **Elbow** is in hand.

### `reachableFrom(start, pid)`  ← updated
- Allows moving onto an **opponent’s corner** **even if the owner’s token is on it**.
- Implementation: treat that corner as a **terminal reachable destination** (add to results, **do not enqueue further traversal**). All other occupied cells remain blocking.

### Token move resolution (destination click)
- Do **not overwrite** the owner’s corner token when scoring.
- Implementation sketch:
  - Compute `oppId = opponentCornerAt(x,y,P.id)` and `destHasOwner = (oppId != null && board[y][x].token === oppId)`.
  - Clear the source (`from`) token.
  - If `destHasOwner`, **do not set** `board[y][x].token = P.id` (leave owner token intact).
  - Award score and ensure the moving token is removed (it’s either never set, or gets cleared immediately).

### `bottomCard()`
- Increments `skipCount` (all tracks) and, if the **bottomed** card is `Elbow`, increments `elbowSkipCount` and calls `checkForceElbow(P)` **before** `updateHUD()`.

### `endTurnImmediate()`
- If ending with a track in hand, increments `skipCount`.
- If ending with an **Elbow** in hand, increment `elbowSkipCount` and call `checkForceElbow(P)`.

### `updateHUD()`
- Draw label: when `state.forcePlace && !P.drawn`, show **“Draw: Ready (Forced)”**.
- Button states:
  - `drawBtn`: disabled when holding a card; else respects lock/used (or enabled during forced-draw).
  - `placeBtn`: **disabled** when `!P.drawn`.
  - `bottomBtn`: **disabled** when `!P.drawn || state.forcePlace || (P.forceElbow && P.drawn===Elbow)`.
  - `endTurnBtn`: **disabled** when `state.forcePlace || (P.forceElbow && P.drawn===Elbow)`.
  - `overrideBtn`: visible **only** when `state.forcePlace && P.drawn`; **hidden** when `P.forceElbow && P.drawn===Elbow`.
- Elbow HUD tag: “Elbows Skipped: x/3” with **⚠ Next Elbow must be placed** at 3/3.

---

## Verification scenarios

### Corner occupied by owner — opponent scoring still possible
1. Yellow has a token sitting on **Yellow’s corner**.  
2. Red moves a token along connected pipes to that cell.  
3. Red’s move list includes Yellow’s corner **even though it’s occupied**.  
4. Red clicks it → Red **scores**; the **Yellow token remains** on the corner.

### Elbow bottoms to threshold → next Elbow must be placed
- Bottom **Elbow** three times; next **Elbow** drawn must be placed (no End Turn/Bottom/Override).

### Three general track skips → forced Draw+Place
- Hit **3/3 Skipped**; Draw is **Ready (Forced)**, End Turn disabled until placing a track (or Override if a card is in hand).

---

## CHANGELOG

### 2025-09-23
- **Corner non-blocking rule:** Owner’s corner token no longer blocks opponents from moving onto that corner to score; owner’s token remains.

### 2025-09-22
- **Elbows Skipped (3/3) forced placement:** Counter & flag; next Elbow must be placed; no Bottom/End Turn/Override while forced.
- **Place button behavior:** Enabled only when a track card is in hand.
- **Forced Draw at 3/3 track skips:** Draw enabled with “Ready (Forced)” when no card; End Turn disabled; Override hidden until a card is drawn.
- **HUD/Buttons:** Bottom disabled when `!P.drawn || state.forcePlace` (track rule) or when elbow-forced is active on an Elbow in hand.
- **RS/RE Rotation:** Confirm-first commit of preview rotation before occupancy checks.
- **RC (Replaceable Cross):** Replace any existing track with a player-owned Cross.
- **Block removal:** Eliminated Block track logic and UI.

### 2025-09-20
- HUD **Skipped** counter with “⚠ Maximum skipped” at 3/3.

### 2025-09-16
- Initial **RS/RE rotation** behavior fixes and confirm-commit rule added.

---

## Contributor checklist

- Always call `checkForcePlace()` **before** `updateHUD()` after any action that might alter `skipCount` or hand state.
- Track elbows via `elbowSkipCount`; call `checkForceElbow(P)` after any action that might alter it.
- In `reachableFrom(start, pid)`: allow **opponent corner with owner token** as a **terminal** destination.
- On scoring at an opponent corner that already has the **owner’s token**, **do not overwrite** the owner token; remove only the scoring token.
- Keep **Place disabled** when `!P.drawn`.
- Keep **Bottom disabled** when `!P.drawn || state.forcePlace || (P.forceElbow && P.drawn===Elbow)`.
- Only show **Override** during forced-draw **if** a card is in hand.
