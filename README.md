# RotoRouter — No-Block Edition

+**Build:** 2025-10-01

+This build includes **Save/Load**, the **Board Fully Tracked** lock, the **Dead-Straight Fix** rule, the **Undo/Redo system**, a unified history fix-phase handler, and the new **SVG Gear Mesh** enhancement (with animated rotations synced to track rotations). It also adds updated **token movement rules**, a **no-legal-placement bottom** path (no penalty, but only once per turn), and **player elimination / turn skip** once a player is out of tokens.

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
  - **Draw is marked as Used** for that turn (so you cannot chain multiple Draw→Bottoms).
  - The “Forced Draw/Place” rule is **waived** in this case to avoid deadlocks.
- Normal Bottom behavior (with skip counters and draw lock) still applies when a legal placement exists.

**Summary:** You may either **keep** an unplaceable card until next turn or **Bottom** it once per turn without penalty.  
If you Bottom it, your Draw state is set to **Used** — you must wait for your next turn to draw again.

---

### 9) Player elimination & turn skip (out of tokens)

- Each player has a lifetime total of **3 tokens**.
- When a player has **no tokens on the board** and has **removed all 3** (e.g., via scoring), they are **out** and their turns are **automatically skipped** thereafter.
- The HUD’s “Token: X/3” now shows **tokens remaining to place** (`3 - (removed + on-board)`).

### 10) Auto-end on 3/3 corners + Finished skip + Game Over (bold green)

- The instant a player reaches their **third opponent corner (3/3)** during a token move, their **turn ends automatically** and advances to the next player.
- A player who has already completed **3/3** is **auto-skipped** at the start of their turn (works after Load, too).
- If **no active players remain**, the game enters **Game Over** state and shows  
  **All players have finished — game over.** in **bold green**.

### 11) Token counter now shows **placed** (not remaining)

- HUD “Token: X/3” now displays **tokens placed so far**: `0/3 → 3/3` (instead of remaining).
- Saves record a coherent `tokens` value (= placed), and **legacy loads** backfill counts from the board and `reached`.
- Fixes the fresh-game case where placing the first token showed **2/3** instead of **1/3**.

### 12) Build adjacency from a token’s connected network

- Placement highlights now include **empty cells adjacent to any track that’s connected** to one of your tokens’ current track cells (not just immediate neighbors).
- This enables legal placement on cells that are **two steps away** via connected straights from the token’s position (matches the attached repro).

### 13) Elbows Skipped 3/3 → **force only if possible** (no deadlocks)

- When a player reaches **Elbows Skipped: 3/3**, the **next drawn Elbow must be placed** **only if** there is **≥1 legal Elbow placement right now**.
- If the drawn **Elbow** has **no legal placement**, the **force is automatically waived** for that card; **Bottom** is allowed **without penalty**, and **End Turn** remains enabled.
- On successful Elbow placement: `elbowSkipCount` resets and the force clears.

### 14) Deadlock-prevention waivers refined

- After **Draw**: if the drawn card has **no legal placement**, the forced-place rule is **waived immediately**.
- On entering **Place**: if **no highlights** exist, forced-place is **waived**.
- In the **HUD** refresh: safety checks **re-waive** forced states if the card in hand can’t be placed, and also waive the general force when the player has **no card** and **Draw is locked** (avoids “can’t draw / can’t end turn” traps).

### 15) Fair Bottoming: no skip penalty after meaningful action

- **Bottom** no longer increments **Skipped** (or Elbows Skipped) if the player has already **applied a rotation** and/or **used a token action** this turn.
- Keeps skip counts from inflating on turns where the player actually did something.

### 16) Turn iteration & stack-safe Game Over guard

- `startTurn()` now **iterates** to the next eligible player (no recursion) and detects **Game Over** when everyone is **finished/out**, preventing call-stack overflows on endgame.

### 17) Main Menu (separate HTML) — setup + live save

- New file: **`MainMenu.html`** (kept separate from the game).
- Choose **board size**, enter **player names**, and mark players as **AI**.
- Saves configuration to **`localStorage`** key `rr.setup` on every input change.
- Click **Start Game** to launch; optional auto-start via `MainMenu.html?go=1`.
- Start button redirects to `RotoRouter.html?start=1` (small cache-buster).

### 18) Game boot reads setup _inside_ the game (IIFE) and honors 7×7/9×9 immediately

- `RotoRouter.html` now loads `rr.setup` **inside** the main script on `DOMContentLoaded`,
  assigns it to `state.setup`, syncs the sidebar **Board Size** select, then calls `newGame()`.
- Ensures **7×7** selection applies immediately without any 9×9 flash.

### 19) In-game **Main Menu** button (same row as “New Game”)

- Added **Main Menu** button next to **New Game** in the **Setup** panel.
- On click: confirmation → navigates to `MainMenu.html`.

### 20) Immediate canvas rescale when board size changes

- After `newGame()` builds a new board, we force an immediate **resize** with a
  double `requestAnimationFrame()` to settle layout before measuring/scale.
- Covers switching **9×9 ↔ 7×7** and menu-launched games.

### 21) Setup helpers

- `loadSetupFromStorage()` reads/normalizes `rr.setup` (board size + players/AI).
- Sidebar **Board Size** dropdown is kept visually in sync with `state.setup.boardSize`.

### 22) New track types: **T** and **RT (Rotatable T)**

- **T** = 3-way junction. Base openings: **E, W, N** (closed S). Rotations cover the other three T shapes.
- **RT** rotates an existing **T** in-place (analogous to **RS/RE**).
- Rendering: T draws a horizontal bar + an upward stem (matches base openings). Rotation preview (Q/E) and ghost behave like other tracks.
- Connectivity/path checks use `OPENINGS.T`, so BFS/reciprocal logic works without special cases.

### 23) Deck updates (with T/RT)

- **7×7:** `Straight: 7, Elbow: 6, Cross: 1, T: 3, RS: 3, RE: 2, RC: 1, RT: 1`
- **9×9:** `Straight: 9, Elbow: 8, Cross: 1, T: 3, RS: 4, RE: 3, RC: 2, RT: 2`
- Notes:
  - Straights reduced by **1 (7×7)** and **2 (9×9)**, then **+1 T** added to both sizes.
  - Total deck sizes preserved versus prior build.

### 24) Placement & rotate/replace updates for T/RT

- **RT** highlights any placed **T**; clicking enters **rotateExisting** and confirms on second click (same flow as RS/RE).
- `computePlaceHighlights()` treats **RS/RE/RT** uniformly when selecting existing tiles to rotate.
- **RC** unchanged (replace/rotate Cross behavior as before).

### 25) Scoring consistency on opponent corners (no ghost tokens)

- When a token **scores** (lands on an opponent’s corner), the mover’s token is:
  - **Always removed** from its **source** cell, and
  - **Never left** on the destination corner.
- Fixes the case where **Score = 1/3** but **3 tokens** still appeared on the board.

### 26) Legacy/Undo safety: prevent over-placement, but show the truth

- Token placement uses an **effective removed** count = `max(tokensRemoved, reached.size)` to block illegal “extra” placements after Undo into old snapshots.
- The **HUD still shows the actual** count (`tokensRemoved + onBoard`) so anomalies remain visible.

### 27) HUD token counter (actual placed + mismatch hint)

### 28) Elbows Skipped 3/3 — Forced Placement

- When a player has skipped **3 Elbows**, the **next Elbow drawn must be placed** if a legal placement exists.
- While this rule is active:
  - **Bottom** button is disabled.
  - **End Turn** button is disabled.
  - Status shows: _“Elbows Skipped: 3/3 ⚠. This elbow must be now be placed”_.
- If no legal placement exists for that Elbow, the force is waived automatically to prevent deadlocks (the Elbow may be Bottomed without penalty).
- On successful Elbow placement, the counter resets and normal play resumes.

### 29) 3 Skipped Track Cards — Force on Next Turn

- After a player skips **3 track cards** (not limited to Elbows), they are forced to **Draw and Place** on their **next turn**.
- **Bottom** and **End Turn** are disabled until the forced card is placed.
- If no legal placement exists for the drawn card, the force is waived automatically.
- Consistent with the Elbow skip behavior, ensuring both rules work uniformly.

### 30) Dead-Straight Fix Refinement

- Trapped Straights are detected only when surrounded by **perpendicular Straights**:
  - Inner cell: all 4 orthogonal neighbors are perpendicular Straights.
  - Border cell: 3 neighbors, all perpendicular.
  - Corner cell: never qualifies.
- At the start of a turn, any trapped Straight may be replaced (free) with a Cross.
- Tokens remain in place during the replacement.
- Undo/Redo and Load resume Dead-Straight Fix prompts correctly.

### 31) Corner Fix Enforcement

- Detects when a player’s **own corner** is sealed by perpendicular Straights.
- The corner must be replaced with a Cross (free) before normal actions continue.
- HUD displays a persistent red warning during this fix phase.
- Integrated with Undo/Redo and Save/Load — the fix prompt reappears when needed.

### 32) HUD / Button State Consistency

- **End Turn** and **Bottom** are now disabled in all forced-placement cases (Elbow skips, 3 skips).
- Sidebar tags (`Skipped`, `Elbows Skipped`) update dynamically as conditions change.
- Warnings are displayed consistently in red with the ⚠ symbol.
- Prevents situations where buttons were active but only blocked by a toast message.

### 33) Rotation Animation Timing (Unified Constants)

- Added two top-level constants to control animation speeds:
  - **`RR_APPLY_TWEEN_MS`** — duration (ms) for full-board Apply rotations (tiles + gears).
  - **`PREVIEW_ROT_TWEEN_MS`** — duration (ms) for Q/E ghost preview rotations during placement.
- All Apply tweens (tiles + gear pulses) now reference `RR_APPLY_TWEEN_MS`, so adjusting one knob changes the whole animation.
- The quick preview spin remains controlled by `PREVIEW_ROT_TWEEN_MS`.

### 34) Safe Placement During Ghost Rotation

- Fixed bug where clicking while a Q/E rotation animation was still in progress could place a track at a **non-orthogonal angle**.
- New behavior:
  - If a click occurs during a ghost tween, the action is **queued** until the animation completes.
  - The placement then snaps to the tween’s exact 90° destination angle, guaranteeing orthogonal alignment.
- This applies both to:
  - **New placements** (placing on an empty cell), and
  - **Rotate-existing commits** (RS/RE/RT).
- Prevents “half-rotated” tracks and ensures consistent appearance.

### 35) Gear artwork update

- Background gear asset switched to **gear-grayblue-03.svg**.
- Added a small cache-buster on load to avoid stale art after updates.

---

## Updated Deck (with T/RT) — supersedes older counts

- **7×7:** `Straight 7, Elbow 6, Cross 1, T 3, RS 3, RE 2, RC 1, RT 1`
- **9×9:** `Straight 9, Elbow 8, Cross 1, T 3, RS 4, RE 3, RC 2, RT 2`

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
  - Marks `drawUsed = true` so the player cannot draw again until their next turn.
- This ensures **only one Bottom per turn** is possible.
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
