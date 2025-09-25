# RotoRouter — No-Block Edition

**Build:** 2025-09-25

This build adds **Save/Load**, the **Board Fully Tracked** lock, and the **Dead-Straight Fix** rule. It also notes a few **pre-existing bugs** that were corrected while integrating these features.

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
  **RR_sample_full_board_9x9.json** (use the copy shared in this chat).

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

## Pre-existing bugs fixed during integration

> These issues existed prior to today’s feature additions and were corrected while wiring up Save/Load and the lock logic.

1. **Load dialog reported “Not a valid RotoRouter save file.”**  
   Cause: a stray `applySnapshot();` self-call inside `applySnapshot(snap)` threw an exception after a successful parse.  
   Fix: removed the self-call; loader now applies the snapshot cleanly.

2. **Corner-seal (“Corner Fix”) only triggered on the *next* player’s turn after a Load.**  
   Cause: turn-start checks weren’t re-run immediately after applying a snapshot.  
   Fix: after `applySnapshot(snap)`, we call `startTurn()` so Corner Fix can trigger right away when needed.

3. **Draw/Place remained enabled immediately after loading a full-board save.**  
   Cause: board-saturation wasn’t recomputed early enough during load.  
   Fix: call `refreshBoardSaturation()` promptly during `applySnapshot(snap)` and then update HUD; Draw/Place/Bottom disable instantly.

> Note: We intentionally did **not** list fixes created as part of *today’s* new features; those will be documented alongside their feature commits when you push this build.

---

## Deck recommendations (unchanged)

### 9×9
- Straight **13**, Elbow **8**, RS **4**, RE **3**, Cross **2**, RCross **2** — total **32**

### 7×7
- Straight **10**, Elbow **6**, RS **3**, RE **2**, Cross **2**, RCross **1** — total **24**

---

## Run
Open **`RotoRouter.html`** in a modern desktop browser. Hard-refresh (Ctrl/Cmd+Shift+R) after swapping files.
