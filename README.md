# RotoRouter — No-Block Edition

**Build:** 2025-09-26

This build includes **Save/Load**, the **Board Fully Tracked** lock, the **Dead-Straight Fix** rule, and the **Undo/Redo system**. It also **unifies history fix‑phase handling** so that **Undo** and **Redo** both re-enter Corner Fix (if sealed) or Dead‑Straight Fix (if available) consistently.

---

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

---

## Deck recommendations

### 9×9

- Straight **13**, Elbow **8**, RS **4**, RE **3**, Cross **2**, RCross **2** — total **32**

### 7×7

- Straight **10**, Elbow **6**, RS **3**, RE **2**, Cross **2**, RCross **1** — total **24**

---

## Run

Open **`RotoRouter.html`** in a modern desktop browser. Hard-refresh (Ctrl/Cmd+Shift+R) after swapping files.
