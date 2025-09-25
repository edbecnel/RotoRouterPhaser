# RotoRouter — No-Block Edition

**Build:** 2025-09-25

This build includes **Save/Load**, the **Board Fully Tracked** lock, the **Dead-Straight Fix** rule, and a new **Undo/Redo system**.

---

## New features

### 1) Save / Load (local JSON)
- **Save Game** downloads a JSON snapshot of the entire game state (board, rotations, ownership, tokens, per-player decks/hand/counters, turn, corner-seal bookkeeping).
- **Load Game** restores the snapshot; transient UI/animations are reset so play resumes cleanly.
- File schema: `__rr_version`, `savedAt`, `N`, `current`, `players[]`, `board[y][x]`, `cornerLockTurns[]`, `cornerSealed[]`.

**How to use**
1. Click **Save Game** in the sidebar to download a `.json` file.
2. Click **Load Game** later and pick that file. Board size switches automatically to match the save.

**Sample file for testing**
- 9×9 with every cell already containing a track:  
  **RR_sample_full_board_9x9.json**

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
  *“Dead-Straight Fix: replace ONE highlighted Straight with a Cross (free).”*
- Click a highlighted cell to apply the fix; normal actions then resume.

---

### 4) Undo / Redo system
Two scopes are supported:

- **Global Undo / Redo**  
  - Steps backward/forward across **any player’s turn**.  
  - Restores the full game state (board, tokens, decks, turn ownership, etc.).  
  - After undo/redo, automatic re-entry into fix phases (Corner Fix, Dead-Straight Fix) occurs if applicable.  

- **Turn Undo / Redo**  
  - Limited to the **current player’s turn only**.  
  - Starts with a baseline snapshot at the beginning of the turn.  
  - Lets a player roll back actions they’ve made within that turn, without affecting prior turns.

**UI**
- **Undo (Global)** / **Redo (Global)** buttons control the cross-turn history.  
- **Undo (Turn)** / **Redo (Turn)** buttons control within-turn rollbacks.  
- Buttons auto-disable when no valid history exists.

---

## Developer notes

- **Undo/Redo config:**  
  ```js
  const UNDO_CONFIG = { enabled: true, global: true, turn: true };
  ```
  Can be toggled in code (no UI).

- **Snapshot tagging:**  
  Each history snapshot carries a `__turnId` so global operations can stay in sync with per-turn history.

- **Dead-Fix integration:**  
  - Undo of a Dead-Straight Fix → restores the straight and re-enters fix mode (prompt/highlights).  
  - Redo of a Dead-Straight Fix → restores the cross and resumes normal play.  
  - Only one free dead-fix is allowed per turn (`deadFixDoneThisTurn` tracked in state and snapshots).

- **Corner Fix integration:**  
  - On redo, if a corner is sealed at the start of turn, the Corner Fix prompt reappears.  
  - On undo, returning to a state before the fix resumes the prompt.

- **Save/Load vs history:**  
  - `snapshotState()` excludes undo/redo stacks.  
  - Save/Load clears history stacks and starts fresh.  
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
