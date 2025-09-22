# RotoRouter — No-Block Edition

**Built:** 2025-09-22

This build removes all support for **Block** tracks and adds several gameplay and HUD fixes to eliminate soft-locks around the skip limit. It also introduces the **RC (Replaceable Cross)** utility card.

---

## Quick start

- Open **`RotoRouter.html`** in any modern desktop browser.
- Recommended: hard-refresh (Ctrl/Cmd+Shift+R) after swapping files so your browser doesn’t cache an older build.

---

## What changed (high level)

- **Block tracks removed** from rules, decks, and UI (non-destructive to the rest of gameplay).
- **Forced Draw at 3/3 skips**: after three skipped placements in a row, the player must **Draw + Place** before ending their turn.
- **HUD improvements**
  - “Skipped: x/3” counter with “⚠ Maximum skipped” at 3/3.
  - **Draw: Ready (Forced)** label appears when a player is forced to draw and has no card in hand.
- **Button-state guards**
  - **Draw**: enabled under forced state even if previously locked/used; never marked “Used” while forced.
  - **Place**: **enabled only when a track card is in hand** (disabled when `!P.drawn`).
  - **Bottom**: disabled when you **don’t** have a card **or** when you’re in the **forced-draw** state.
  - **End Turn**: disabled while forced; **Override** only shows if a card is in hand.
- **RS/RE rotation confirm-first**: Clicking the **same** cell when using RS/RE commits the rotated angle **before** occupancy checks, ensuring the rotation “sticks.”
- **New card – RC (Replaceable Cross)**: Replace any existing track with a Cross you own.

---

## Detailed behavior & UX

### Forced Draw (after 3/3 skips)

When a player’s `skipCount ≥ 3` **and** they have **no drawn card**:

- **Draw** is **enabled** and labeled **“Draw: Ready (Forced)”**.
- **End Turn** is **disabled**.
- **Override** is **hidden** (no card to act on yet).
- **Bottom** is **disabled** (no card, and bottoming is irrelevant while forced).
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

### RS/RE rotation confirm-first

Using **RS/RE** on an existing Straight/Elbow:

1) Click the target track to enter rotate-existing subphase.  
2) Use **Q/E** to set the preview angle.  
3) Click the **same** cell again to **commit** that angle **before** any occupancy or adjacency checks.  
4) The RS/RE card is discarded; rotation “sticks.”

### RC (Replaceable Cross) card

- Draw **RC**, select **any existing track**.
- On confirm, the target track is replaced by a **Cross** owned by the acting player.
- Rotation is normalized (Cross is symmetric). Ownership transfers to the actor.
- Forced state clears if this resolves a forced-draw turn.

---

## Developer notes (function-level pointers)

> Names below refer to functions/logic you’ll find in `RotoRouter.html`. Keep these invariants intact to prevent soft-locks.

### `checkForcePlace()`
- Sets `state.forcePlace = (P.skipCount >= 3)`.
- If forced **and** **no card in hand**: clears `P.drawLocked` and `P.drawUsed` so **Draw** is usable and not marked “Used.”
- Call this **before** rendering the HUD whenever skip count or hand state might change (e.g., at start of turn, after `bottomCard()` increments skip count).

### `bottomCard()`
- Increments `skipCount` and sets `P.drawLocked = true` (normal path).
- **Important ordering**: call `checkForcePlace()` **before** `updateHUD()` so forced-draw is visible immediately (label reads “Ready (Forced)”, End Turn disabled, Bottom disabled).

### `drawCard()`
- Guards for `drawLocked`/`drawUsed` are **waived while forced**.
- While forced, **do not** set `P.drawUsed = true`; keep it **false** so the HUD doesn’t show “Used” and players can proceed to place.

### `updateHUD()`
- Draw label: when `state.forcePlace && !P.drawn`, show **“Draw: Ready (Forced)”**.
- Button states:
  - `drawBtn`: enabled when not holding a card and not locked/used (or when forced).
  - `placeBtn`: **disabled** when `!P.drawn` (enabled only when a card is in hand).
  - `bottomBtn`: **disabled** when `!P.drawn || state.forcePlace`.
  - `endTurnBtn`: **disabled** when `state.forcePlace`.
  - `overrideBtn`: visible **only** when `state.forcePlace && P.drawn`.

### Placement/consumption paths
- On successful placement / confirmed RS/RE / RC:
  - Reset `P.skipCount = 0`.
  - Clear `state.forcePlace = false`.
  - Clear rotate/preview substate, update HUD.

---

## Verification scenarios (step-by-step)

### A) Three bottoms then forced draw
1. Turn 1 (Red): **Draw → Bottom → End Turn** → HUD shows **Skipped: 1/3**.  
2. Turn 2 (Red): **Draw → Bottom → End Turn** → HUD shows **Skipped: 2/3**.  
3. Turn 3 (Red): **Draw** and **End Turn** without placing → still **Skipped: 2/3**.  
4. Turn 4 (Red, forced): **Draw** is **enabled** (**Ready (Forced)**), **End Turn** is **disabled**, **Override** is **hidden**, **Bottom** is **disabled**, **Place** is **disabled**.  
5. **Draw** a card: **Override** becomes **visible**; **Place** becomes **enabled**; **End Turn** still **disabled**.  
6. **Place** the card (or use RS/RE/RC) → force clears, next turn is normal.

### B) Forced state after bottoming on the third skip
1. Turn 1 (Red): **Draw → Bottom → End Turn** → **Skipped: 1/3**.  
2. Turn 2 (Red): **Draw → Bottom → End Turn** → **Skipped: 2/3**.  
3. Turn 3 (Red): **Draw → Bottom** → **Skipped: 3/3 ⚠ Maximum skipped** now triggers forced state.  
   - **Draw** becomes **enabled** with **Ready (Forced)**.  
   - **Place** and **Bottom** are **disabled**; **End Turn** is **disabled**; **Override** hidden.  
4. **Draw** then **Place** to clear the force.

### C) RS/RE rotation confirm
1. Draw **RS** (or **RE**).  
2. Click an existing Straight/Elbow → press **Q/E** to set angle.  
3. Click the **same** tile → rotation commits; card is discarded.

### D) RC replacement
1. Draw **RC**.  
2. Hover highlights show valid track targets.  
3. Click a target → it becomes a **Cross** owned by the current player; card is discarded.  
4. If under forced-draw, this resolves the force.

---

## CHANGELOG

### 2025-09-22
- **Place button behavior**
  - **Enabled only when a track card is in hand** (`!P.drawn` disables the button).
- **Forced Draw at 3/3 skips**
  - Draw enabled with **“Ready (Forced)”** when no card in hand.
  - **End Turn** disabled while forced; **Override** hidden until a card is drawn.
  - Forced draws no longer mark Draw as “Used.”
- **HUD/Buttons**
  - **Bottom** disabled when `!P.drawn || state.forcePlace`.
  - Clearer status messages when force triggers.
- **RS/RE Rotation**
  - Confirm-first commit of preview rotation before occupancy checks.
- **RC (Replaceable Cross)**
  - Replace any existing track with a player-owned Cross.
- **Block removal**
  - Eliminated Block track logic and UI.

### 2025-09-20
- HUD **Skipped** counter with “⚠ Maximum skipped” at 3/3.

### 2025-09-16
- Initial **RS/RE rotation** behavior fixes and confirm-commit rule added.

---

## Contributor checklist

- Always call `checkForcePlace()` **before** `updateHUD()` after any action that might alter `skipCount` or hand state.
- While forced, **allow Draw** even if previously locked/used, and **never** mark it “Used” until a placement resolves the force.
- Keep **RS/RE confirm-first** logic **above** occupancy checks.
- Keep **Place disabled** when `!P.drawn`.
- Keep **Bottom disabled** when `!P.drawn || state.forcePlace`.
- Only show **Override** during forced state **if** a card is in hand.
