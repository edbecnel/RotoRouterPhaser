# RotoRouter — No-Block Edition

+**Build:** 2025-09-27

- This build includes **Save/Load**, the **Board Fully Tracked** lock, the **Dead-Straight Fix** rule, the **Undo/Redo system**, a unified history fix-phase handler, and the new **SVG Gear Mesh** enhancement (with animated rotations synced to track rotations). It also adds updated **token movement rules**, a **no-legal-placement bottom** path (no penalty), and **player elimination / turn skip** once a player is out of tokens.

  ***

## New features

### 1) Save / Load (local JSON)

- **Save Game** downloads a JSON snapshot of the entire game state (board, rotations, ownership, tokens, per-player decks/hand/counters, turn, corner-seal bookkeeping).
- **Load Game** restores the snapshot; transient UI/animations are reset so play resumes cleanly.
- File schema: `__rr_version`, `savedAt`, `N`, `current`, `players[]`, `board[y][x]`, `cornerLockTurns[]`, `cornerSealed[]`.

**How to use**

1. Click **Save Game** in the sidebar to download a `.json` file.
2. Click **Load Game** later and pick that file. Board size switches automatically to match the save.

**Sample files for testing**

- 9×9 with every cell already containing a track: **RR_sample_full_board_9x9.json**
- 9×9 with a **sealed corner** and multiple **trapped straights** ready for Dead‑Straight Fix: **RR_sample_corner_and_deadfix_9x9.json**

---

### 2) Board Fully Tracked — placement lock

When **every cell on the board has a track**, track placement is locked:

- **Draw / Place / Bottom** are **disabled**.
- **Forced-Draw** (3 general skips) and **Elbow-force** (3 elbow skips) are **suppressed** while locked.
- **Token Action** and **Roll / Apply** remain available.

Lock detection runs on **New Game**, **Load**, and after any topology change (place, RC/RS/RE, Apply). The HUD shows **“Draw: Locked (Board Full)”** and disables the track-placement buttons.

---

### 3) Dead-Straight Fix (rule)

A **trapped straight** is a **Straight** that is surrounded by **perpendicular Straights**:

- **Inner cell:** all **4** orthogonal neighbors exist and are perpendicular Straights.
- **Border cell:** exactly **3** in-bounds neighbors exist and all 3 are perpendicular Straights.
- **Corner cell:** only 2 neighbors → **never** qualifies.

At the **start of your turn**, if any trapped straights exist:

- You may replace **exactly one** highlighted trapped straight with a **Cross** (free).
- Tokens on that cell **remain**.
- If trapped straights remain, each subsequent player may fix **one** on their turn until none remain.
- Works whether or not the board is fully tracked.

**UI flow**

- Turn start highlights all eligible cells and prompts:  
  _“Dead-Straight Fix: replace ONE highlighted Straight with a Cross (free).”_
- Click a highlighted cell to apply the fix; normal actions then resume.

---

### 4) Undo / Redo system

Two scopes are supported:

- **Global Undo / Redo**

  - Steps backward/forward across **any player’s turn**.
  - Restores the full game state (board, tokens, decks, turn ownership, etc.).
  - After undo/redo, automatic re-entry into fix phases (**Corner Fix**, **Dead‑Straight Fix**) occurs if applicable.

- **Turn Undo / Redo**
  - Limited to the **current player’s turn only**.
  - Starts with a baseline snapshot at the beginning of the turn.
  - Lets a player roll back actions they’ve made within that turn, without affecting prior turns.

**UI**

- **Undo (Global)** / **Redo (Global)** buttons control the cross-turn history.
- **Undo (Turn)** / **Redo (Turn)** buttons control within-turn rollbacks.
- Buttons auto-disable when no valid history exists.

---

### 5) Corners Score Table

The sidebar now includes a dedicated **Corners Score Table**:

- **Columns:** Color (with swatch), Quadrant (with coordinates, e.g. `NE (8,0)` on a 9×9), and Score.
- **Score format:** Matches the Token Score tag (`2/3 [R, G]`), showing how many opponent corners have been reached and by which players.
- Automatically updates after every token scoring event or Undo/Redo.
- Replaces the earlier legend/separator layout, consolidating all corner/score info into a single clear table.

---

### 6) SVG Gear Mesh (with animated rotation)

- Each cell of the board now renders a **gear image** (`gear-grayblue.svg`) behind tracks/tokens.
- Neighboring gears alternate **0° / 90° base orientation** so the teeth appear to mesh visually.
- On **Apply Rotation** (CW90, CCW90, or 180°):
  - Gears animate at the **same duration/easing** as track rotation tweens.
  - Each gear maintains a **persistent offset** so Undo/Redo and Save/Load restore orientations correctly.
- Gears do **not** spin on RS/RE actions, only on Apply die rolls.
- A fallback vector gear still draws if the SVG fails to load.

**Assets**

- Place `gear-grayblue.svg` in the same folder as `RotoRouter.html`.
- Adjust `GEAR_OUTER_FIT` in code (default `1.00`) to tweak visual spacing between meshed gears.

---

---

### 7) Token movement rules (blocking + corner exception)

- Tokens **cannot pass through any other tokens** by default.
- If a player’s token would otherwise be **trapped by opponents’ tokens**, movement may **pass through opponent tokens** (never through your own) solely to find legal destinations. You still **cannot end** on an occupied non-corner cell.
- **Corners can’t be blocked:** you may **move onto an opponent’s corner** even if their token already occupies that corner; that move **scores** and resolves per normal rules.

---

### 8) No-legal-placement → Bottom without penalty (and no forced draw deadlock)

- When the drawn **track card** has **zero legal placements** anywhere on the board:
  - **Bottom**ing that card moves it to the bottom of the deck **without** incrementing **Skipped** or **Elbows Skipped**.
  - **Draw is not locked**; you may draw again immediately in the same turn.
  - The “Forced Draw/Place” rule is **waived** in this case to avoid deadlocks.
- Normal Bottom behavior (with skip counters and draw lock) still applies when a legal placement exists.

---

### 9) Player elimination & turn skip (out of tokens)

- Each player has a lifetime total of **3 tokens**.
- When a player has **no tokens on the board** and has **removed all 3** (e.g., via scoring), they are **out** and their turns are **automatically skipped** thereafter.
- The HUD’s “Token: X/3” now shows **tokens remaining to place** (`3 - (removed + on-board)`).

## Developer Notes

### Undo/Redo config

```js
const UNDO_CONFIG = { enabled: true, global: true, turn: true };
```

Can be toggled in code (no UI).

### Snapshot tagging

Each history snapshot carries a `__turnId` so global operations can stay in sync with per-turn history.

### Fix-phase integration after Undo/Redo (unified)

- A single helper now manages re-entry into fix phases after **either** Undo or Redo:
  ```js
  // unified entry point used by applySnapshot(..., { fromHistory: true })
  function maybeEnterFixPhasesAfterHistory(op) {
    /* op is reserved (unused) */
  }
  ```
- **Priority:** `Corner Fix` (if your corner is sealed) → else `Dead‑Straight Fix` (if available and not consumed this turn).
- The previous helper `maybeResumeDeadFixAfterHistory()` has been **replaced** by `maybeEnterFixPhasesAfterHistory(...)`.
- The legacy name `maybeEnterFixPhasesAfterRedo()` has been **removed** (it was redundant once the unified helper existed).  
  If you need back-compat for external callers, you can add a 1‑line alias:
  ```js
  function maybeEnterFixPhasesAfterRedo() {
    return maybeEnterFixPhasesAfterHistory();
  }
  ```

### Dead‑Straight Fix bookkeeping

- Undo of a Dead‑Straight Fix → restores the straight and **re-enters** the fix subphase (prompt/highlights).
- Redo of a Dead‑Straight Fix → restores the cross and resumes normal play.
- Only **one** free dead-fix per turn (`deadFixDoneThisTurn` in state & snapshots).

### Corner Fix bookkeeping

- On redo, if a corner is sealed at the start of turn, the Corner Fix prompt reappears.
- On undo, returning to a state before the fix resumes the prompt.

### Save/Load vs history

- `snapshotState()` excludes undo/redo stacks.
- Save/Load **clears** history stacks and starts fresh.
- Undo/Redo calls pass `{ fromHistory:true }` so stacks aren’t cleared and fix subphases are restored properly.

### Gear offset persistence

- Each cell now carries a persistent `gearOffset` (degrees) so visual gear orientations match track rotations.
- `snapshotState()` includes `gearOffset`, and `applySnapshot()` restores it.
- This ensures **Undo/Redo** and **Save/Load** both correctly restore gear orientations, eliminating drift or snap-back.

### Token pathfinding (strict + fallback)

- `reachableFrom()` runs in **two phases**:
  1. **Strict:** treats **any token** as a blocker (no pass-through).
  2. **Fallback (only if trapped):** allows pass-through over **opponent tokens** (never your own) to find destinations. You still can’t end on an occupied non-corner cell.
- **Corner exception**: If a neighbor is an opponent’s **own corner** and their token is on it, it remains a **legal terminal** (corners can’t be blocked) but we **don’t traverse beyond** it.

### No-legal-placement bottoming (no penalty)

- `canPlaceDrawnNow()` checks if the current drawn track has any legal placements.
- If **none**, `bottomCard()`:
  - Moves the card to the bottom of the deck,
  - **Does not** increment `skipCount` or `elbowSkipCount`,
  - **Does not** lock draw and clears `drawUsed` so the player may draw again.
- `checkForcePlace()` waives the forced-draw rule while holding an **unplaceable** card to prevent deadlocks.

### Player elimination / skipping turns

- Each player tracks `tokensRemoved` (lifetime removed tokens).
- At `startTurn()`, if `tokensRemoved >= 3` **and** `countTokens(pid) === 0`, that player is **skipped** automatically.
- The HUD “Token: X/3” displays remaining supply (`3 - (removed + on-board)`).

### Legacy save backfill (pre-`tokensRemoved`)

- Old saves won’t contain `tokensRemoved`. On `applySnapshot()`:
  - We infer a sensible value using the player’s `reached` set and current on-board token count.
  - If **no tokens are on board** and no other info is available, we conservatively assume the player has exhausted supply so the **skip-out** logic works after load.

---

## Deck recommendations

### 9×9

- Straight **13**, Elbow **8**, RS **4**, RE **3**, Cross **2**, RCross **2** — total **32**

### 7×7

- Straight **10**, Elbow **6**, RS **3**, RE **2**, Cross **2**, RCross **1** — total **24**

---

## Run

Open **`RotoRouter.html`** in a modern desktop browser. Hard-refresh (Ctrl/Cmd+Shift+R) after swapping files.
