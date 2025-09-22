# RotoRouter — No-Block Edition

**Built:** 2025-09-22

This build removes all support for **Block** tracks and adds gameplay/HUD fixes to eliminate soft-locks around skip limits. It also introduces a new rule to **force the introduction of elbows** after players skip them three times.

---

## Quick start

- Open **`RotoRouter.html`** in any modern desktop browser.
- Recommended: hard-refresh (Ctrl/Cmd+Shift+R) after swapping files so your browser doesn’t cache an older build.

---

## What changed (high level)

- **Block tracks removed** from rules, decks, and UI (non-destructive to the rest of gameplay).
- **Forced Draw at 3/3 track skips**: after three skipped track placements in a row, the player must **Draw + Place** before ending their turn.
- **NEW — Elbows Skipped (3/3) forced placement**
  - Tracks how many times a player **skips placing an Elbow** (either by bottoming an **Elbow** or ending their turn while holding an **Elbow**).
  - After **3** elbow skips, the player is **flagged**: the **next time they draw an Elbow**, they **must place it** that turn.
  - While holding that Elbow under the elbow-forced state: **End Turn** and **Bottom** are **disabled**; **Place** is enabled. No override.
  - After the Elbow is placed, the elbow skip counter resets to **0** and the elbow-forced flag clears.
- **HUD improvements**
  - “Skipped: x/3” counter with “⚠ Maximum skipped” at 3/3 (track placement rule).
  - **NEW:** “Elbows Skipped: x/3” counter with “⚠ Next Elbow must be placed” at 3/3.
  - **Draw: Ready (Forced)** label appears when a player is forced to draw and has no card in hand.
- **Button-state guards**
  - **Draw**: enabled under forced-draw even if previously locked/used; never marked “Used” while forced-draw.
  - **Place**: **enabled only when a track card is in hand** (`!P.drawn` disables the button).
  - **Bottom**: disabled when you **don’t** have a card, when **forced-draw** is active, or when **elbow-forced** is active on an Elbow in hand.
  - **End Turn**: disabled while **forced-draw**, and also when **elbow-forced** is active on an Elbow in hand.
  - **Override**: only visible during **forced-draw** when a card is in hand; **hidden** during **elbow-forced** (no bypass).
- **RS/RE rotation confirm-first**: Clicking the **same** cell when using RS/RE commits the rotated angle **before** occupancy checks, ensuring the rotation “sticks.”
- **RC (Replaceable Cross)**: Replace any existing track with a Cross you own.

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

### NEW — Elbows Skipped (3/3) forced placement

- Each time a player **skips placing an Elbow** (either by **Bottom** on an **Elbow** card **or** ending the turn while **holding** an **Elbow**), increment **`elbowSkipCount`**.
- When **`elbowSkipCount ≥ 3`**, set a **pending elbow-forced** flag for that player.
- The **next time that player draws an Elbow**, they are **forced to place it immediately**:
  - **End Turn** is **disabled**.
  - **Bottom** is **disabled**.
  - **Place** remains **enabled** (and highlights show legal cells as usual).
  - **Override** is **hidden** (cannot bypass elbow forcing).
- After the Elbow is successfully placed:
  - Reset `elbowSkipCount = 0` and clear the elbow-forced flag.

> Note: The elbow-forced flag **persists across turns and non-elbow draws**; it only clears once the player places an Elbow.

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
- Forced-draw clears if this resolves a forced-draw turn.

---

## Developer notes (function-level pointers)

> Names below refer to functions/logic in `RotoRouter.html`. Keep these invariants intact.

### `checkForcePlace()` (track skips)
- Sets `state.forcePlace = (P.skipCount >= 3)`.
- If forced **and** **no card in hand**: clears `P.drawLocked` and `P.drawUsed` so **Draw** is usable and not marked “Used.”
- Call this **before** rendering the HUD whenever skip count or hand state might change.

### `checkForceElbow(P)` (elbow skips)
- Sets `P.forceElbow = (P.elbowSkipCount >= 3)`; persists across turns.
- Does **not** change draw locks—elbow forcing only applies when an **Elbow** is in hand.

### `bottomCard()`
- Increments `skipCount` (track rule) and, **if the bottomed card is `TrackCard.Elbow`**, increments `elbowSkipCount` and calls `checkForceElbow(P)` **before** `updateHUD()`.

### `endTurnImmediate()`
- If ending with a track card in hand, increments `skipCount` (existing behavior).
- **NEW:** If ending with `TrackCard.Elbow` in hand, increment `elbowSkipCount` and call `checkForceElbow(P)`.

### `drawCard()`
- Forced-draw guards are waived while `state.forcePlace` (existing behavior).
- **After drawing:** if `P.forceElbow` **and** `P.drawn === TrackCard.Elbow`, set a status like **“Elbows Skipped reached 3/3 — you must place this Elbow now.”**

### `updateHUD()`
- Draw label: when `state.forcePlace && !P.drawn`, show **“Draw: Ready (Forced)”**.
- Button states:
  - `drawBtn`: disabled when holding a card; else respects lock/used (or enabled during forced-draw).
  - `placeBtn`: **disabled** when `!P.drawn`.
  - `bottomBtn`: **disabled** when `!P.drawn || state.forcePlace || (P.forceElbow && P.drawn===TrackCard.Elbow)`.
  - `endTurnBtn`: **disabled** when `state.forcePlace || (P.forceElbow && P.drawn===TrackCard.Elbow)`.
  - `overrideBtn`: visible **only** when `state.forcePlace && P.drawn`; **hidden** when `P.forceElbow && P.drawn===TrackCard.Elbow`.
- Elbow HUD tag:
  - Update `Elbows Skipped: x/3`, and when `x ≥ 3` add **“⚠ Next Elbow must be placed”**.

### Placement/consumption paths
- On any successful placement / confirmed RS/RE / RC: reset `P.skipCount = 0` and clear `state.forcePlace = false` (existing behavior).
- **Additionally:** when the placed track is an **Elbow**, reset `P.elbowSkipCount = 0` and `P.forceElbow = false`.

---

## Verification scenarios

### A) Hitting the elbow threshold with Bottom
1. Turn 1 (Red): Draw **Elbow** → **Bottom** → **Elbows Skipped: 1/3**.  
2. Turn 2 (Red): Draw **Elbow** → **Bottom** → **Elbows Skipped: 2/3**.  
3. Turn 3 (Red): Draw **Elbow** → **Bottom** → **Elbows Skipped: 3/3 ⚠ Next Elbow must be placed**.  
4. Later, when Red next draws an **Elbow**: **End Turn/Bottom disabled**, **Place enabled** → must place Elbow; afterwards `elbowSkipCount` resets to **0**.

### B) Hitting the elbow threshold by simply skipping placement
1. Turn 1: Draw **Elbow** → **End Turn** (keep in hand) → **Elbows Skipped: 1/3**.  
2. Turn 2: **End Turn** again (still holding Elbow) → **Elbows Skipped: 2/3**.  
3. Turn 3: **End Turn** again → **Elbows Skipped: 3/3 ⚠ Next Elbow must be placed**.  
4. When Red eventually **Bottoms** or **Draws** into an **Elbow**: forced to place that Elbow.

### C) Interaction with forced-draw (track skips)
- If both rules are in effect simultaneously (e.g., forced-draw active and elbow-forced pending), the button guards remain consistent: **End Turn** disabled and **Bottom** disabled in both cases when the relevant condition is met.

---

## CHANGELOG

### 2025-09-22
- **Elbows Skipped (3/3) forced placement**
  - Count increments when an **Elbow** is bottomed **or** when a turn is ended while **holding** an **Elbow**.
  - After 3, the next **Elbow** drawn **must be placed** that turn (no Bottom, no End Turn, no Override).
  - Elbow counter and flag clear upon placing an Elbow.
- **Place button behavior**
  - **Enabled only when a track card is in hand** (`!P.drawn` disables the button).
- **Forced Draw at 3/3 track skips**
  - Draw enabled with **“Ready (Forced)”** when no card in hand.
  - **End Turn** disabled while forced; **Override** hidden until a card is drawn.
  - Forced draws no longer mark Draw as “Used.”
- **HUD/Buttons**
  - **Bottom** disabled when `!P.drawn || state.forcePlace` (track rule) or when elbow-forced is active on an Elbow in hand.
  - Clearer status messages when either force triggers.
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
- Track elbows via `elbowSkipCount`; call `checkForceElbow(P)` after any action that might alter it (Bottom-on-Elbow, End Turn with Elbow in hand).
- While **forced-draw**, allow Draw even if previously locked/used, and **never** mark it “Used” until a placement resolves the force.
- While **elbow-forced and Elbow in hand**, **disable End Turn and Bottom**; hide Override.
- Keep **RS/RE confirm-first** logic **above** occupancy checks.
- Keep **Place disabled** when `!P.drawn`.
- Keep **Bottom disabled** when `!P.drawn || state.forcePlace || (P.forceElbow && P.drawn===TrackCard.Elbow)`.
- Only show **Override** during forced-draw **if** a card is in hand.
