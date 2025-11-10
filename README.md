# RotoRouter

A tactical route-building board game with optional AI players.

---

---

### What’s new

## What’s new (Nov 10, 2025)

### Turn counter (per player)

- Each player now has a **turns** counter that **increments at the start of a new turn only when there are no per-turn Redos available**. This still counts even if only one player remains. Undo/Redo do not change the count.
- **Save/Load:** `players[].turns` is persisted in snapshots and restored on load; older saves backfill the value to `0`.
- **UI:** In the **Corners** score table, hovering a player’s score shows a tooltip `Turns: ###`, and clicking the score shows a toast with the same text.
- **Look:\*** Game tokens have new glossy 3D appearance (made from rendered PNG images encoded at base64 and embedded in the source file)

### AI Only Remaining Pause

- Now when there are only one or more AI players left in the game, the game pauses asking if you want to continue.

### What's new (Oct 26, 2025)

## 2025-10-25 — Cleanup Phase-1 (mechanical fixes)

Summary: fairness + determinism + one null-guard. No UI or layout changes.

## Deterministic deck recycle

When the draw pile empties, the discard pile is moved back to draw without shuffling (FIFO). This preserves the original shuffle order for the entire game.

## AI draw fairness

Removed the legacy “prefer connector cards (RS/RE/RT/RC)” look-ahead. AI now draws exactly like humans: top of the draw pile only.

## Corner-highlight guard

drawTurnCornerHighlight() now skips safely if there’s no current player yet (boot/transition safety). No visual change; eliminates a rare console error.

Files touched: RotoRouter.html

## Sanity checks:

New game → draw/discard until recycle: order remains consistent, no reshuffle.
Early AI turns: no bias toward connectors; card mix reflects true top-of-deck.
Launch from MainMenu: no “reading 'active' of undefined” console error on initial paint.

### What’s new (Oct 16, 2025)

## New: AI Delay Control (1–10 → ×0.5…×5.0)

You can now scale the AI's think-time/delay using a simple slider in the sidebar.

- Where: In the Actions → Token panel, under the “AI debug trace (console)” checkbox.
- Control: AI delay slider from 1 to 10 with a live label showing the multiplier (e.g. ×1.0).
- Range: 1 → ×0.5 (shorter delays, faster AI), … 10 → ×5.0 (longer delays, slower AI).
- Default: 2 (×1.0). This matches the previous timing: aiSleepTime=200, aiSleepTime2=300, aiSleepTime3=280.
- Persistence: The setting is saved per-browser and restored on next load.
- Design note: The control adjusts delay length (not speed); larger values mean longer waits.
- Design note: The UI includes the caption: 1 = shorter (faster), 10 = longer (slower) and a numeric “×multiplier” label.

### Oct 15, 2025

- **Last-placed highlight** – on your turn, the most recent track you placed is outlined with a soft glow so you can quickly find it again.
- **AI move preview** – before the AI commits a placement/rotation, it briefly flashes a ring on the chosen cell (a quick “heads-up” beat).
- **AI debug trace (console)** – new sidebar toggle (“AI debug trace (console)”) prints the AI’s short decision trace (targets considered, scores, and final choice).

## Using the new features

- **Seeing your last placement**
  - After you place a track, a subtle glow/border marks that cell **only on your turn**. It fades when you change context or another track becomes the “last placed”.
- **Watching AI intentions**
  - When the current player is AI, you’ll see a short flash on the cell it’s about to use (whether rotating an existing tile or placing a new one). Then it commits.
- **Debugging AI**
  - In the sidebar, under _Actions → Token_, tick **AI debug trace (console)**. Open the browser console to view: scores for candidate moves, chosen action, and any fallbacks.

### Oct 14, 2025

## Fixed bug related to AI

- When playing 2 players (1 human and 1 AI) or 3 players (1 humand and 2 AI), the "Continue" game state was being incorrectly triggered
  after the human player landed all his tokens. Fixed logic to determine when a player has truly finished.
- Fixed bug whereby the solo-AI "Continue" warning message was getting overriden by "Turn passed." message endTurnImmediate(). Extended the guard
  that suppresses the "Turn passed." message to also suppress it while we're paused for the solo-AI prompt.

### Oct 13, 2025

## New: Token Visibility Toggle (Hide/Unhide)

You can now quickly hide all tokens to view the track layout unobstructed.

- **Where:** Tokens panel in the sidebar.
- **How:** Click **Hide** to hide all tokens; the button switches to **Unhide**.
- **Scope:** Hiding persists for the **rest of your current turn** unless you manually unhide.
- **Auto-unhide:** If you click **Token Action**, tokens are immediately made visible again and the button reverts to **Hide**.
- **Keyboard shortcuts:**
  - **H** — Hide tokens
  - **U** — Unhide tokens

> Note: Hiding affects **visuals only**; it does **not** change reachability or legal move checks.

### AI: rotation gating, exploration, and no-op tolerance

- **Rotation gating:** the AI **only rolls the die** after first probing for a legal token move and only when a rotation is likely to help.
- **Exploratory move before rotating:** if recent rotations from the same cell don’t help, the AI will try a short **exploration move** to a nearby branching spot and re-evaluate before attempting another die roll.
- **No-op tolerance:** if neither a token move nor a rotation would improve connectivity (e.g., early game with no nearby networks), the AI may **Bottom and end the turn** without moving or rotating.
- **Anti-bounce:** remembers the last hop/edge to prevent A↔B “ping-pong”.

### AI pause & Continue

- When there are **onlyy one or more AI players** remaining and all human players are finished, the game **pauses** and shows:
  > “There is only one or more AI players left. If you wish to stop now, click **New Game**, or press **Continue** to proceed.”
- Press **Continue** to resume AI play; the banner and button disappear.

### Forced play / HUD polish

- If **3 elbows were skipped**, the next drawn Elbow must be placed (if legal). When the counter resets to **0/3**, the **warning style clears**.
- If a forced card has **no legal placement**, the force is **waived** to avoid deadlocks.

### Bug fixes

- Fixed “endless rotation” cases by adding stricter rotation gates and an exploration fallback.
- Fixed standings to avoid duplicate “1st place”; unique places advance to **2nd/3rd/4th** correctly.

### Track colors

- Newly placed tracks now adopt the **placing player’s color** for their line art.
- **RC** (Replace with Cross) keeps the **original tile’s color** when replacing an existing tile (the Cross inherits the replaced tile’s color), but uses the **placing player’s color** when placing RC into an empty cell.

### AI: rotation gating & anti-bounce

- **Rotation gating:** the AI **only rolls the die to rotate after first probing for a legal token action**. If a legal move exists, it will not roll pre-emptively. (Prevents rotating immediately after placing a corner T.)
- **Single-rotation per stall cycle:** when stuck, the AI will at most perform **one rotation**, then re-evaluate.
- **Anti-bounce memory:** remembers last hop and a per-turn visited set to avoid A↔B oscillation (“ping-pong”).

### Forced play / HUD polish

- If **3 elbows were skipped**, the next drawn Elbow must be placed (if legal). If that state is cleared or waived, the **warning style is removed** and the counter returns to its normal color.
- If a forced card has **no legal placement right now**, the force is **waived** so play can continue.

### Bug fixes

- Fixed cases where the AI could “ping-pong” between two tracks without ever attempting a rotation.
- Fixed elbow-skip banner not clearing its red/warn styling when the counter reset to **0/3**.

## What’s new (Oct 2025)

### Gameplay/UI

- **Standings panel** at the top of the sidebar shows active players only. Rankings remain hidden until the _first_ scoring event; the first scorer becomes **1st** (bold), all others start as **2nd**. Ties display as **T-1st**, **T-2nd**, etc.
- **Corners table** now includes a **Place** column mirroring the Standings.
- **Dual-use rotation cards:** **RS/RE/RT/RC** can rotate/replace **existing** tiles _or_ be placed as **new** Straight/Elbow/T/Cross, respectively.
- **Deck scaling** for 2–3 active players so the total track supply stays healthy. Per-player minimums:
  - **7×7:** 7 Straight, 6 Elbow, 3 T, 1 Cross
  - **9×9:** 9 Straight, 8 Elbow, 3 T, 1 Cross  
    Rotation cards scale with the deck. Decks are built at **New Game** from board size + active player count.
- **Touch rotation mode:** tap a legal empty cell to preview; use **Q/E** or the ⟲/⟳ buttons to rotate; tap again (or **Confirm**) to place.

### Rules/Edge-cases

- **Forced-play** counters:
  - **3 skipped tracks** → next drawn track must be placed _if_ legal.
  - **3 skipped elbows** → next Elbow drawn must be placed _if_ legal.
  - If a forced card has **no legal placement**, the force is **waived** (no deadlocks).
- **Pass-through corners:** tokens may **pass through** corners they’ve already scored but **cannot stop** on those corners again.
- **Dead-Straight fix:** fully trapped Straights can be converted to Cross (one per turn) at the start of your turn.
- **Corner fix:** if your home corner is sealed, you must Cross it before other actions.

### AI (Robot)

- **Path-seeking movement:** prefers moves that strictly reduce Manhattan distance to any **unreached** opponent corner; avoids immediate back-tracking and short loops.
- **Anti-bounce memory:** remembers the last hop and a per-turn visited set to stop A↔B oscillation.
- **Stall handling:** when no progress is possible, AI prioritizes **die rotations** and conservatively **skips RS/RE/RT/RC**, saving them for endgame connections.
- **Dual-use awareness:** **RT** now mirrors **RS/RE** behavior for placement _and_ rotation.
- **Save/Load:** AI flags, per-turn memory, and counters survive snapshot load.

### AI Phase-2 / Placement Updates (Oct 12 2025)

- **Placement legality tightened:** AI can no longer place tracks that only touch opponent tiles without connecting to them. A placement must connect to at least one friendly track, or to a joined network where both sides are connected.
- **Adjacency refinement:** human players may always place adjacent to their own network (even without connecting); opponents can only place adjacent to yours once their network has become connected to it.
- **Forced-placement state:** if you Undo during a forced turn, the state persists; **Bottom** and **End Turn** stay disabled until the forced card is placed or waived.
- **AI stall recovery:** now performs at most one die-rotation per stall cycle instead of every turn.
- **AI card economy:** saves RS/RE/RT/RC for endgame, Bottoms weak cards only when not forced.
- **Tooltip added:** End Turn shows a short hint when disabled due to forced placement.

### Bug fixes

- Fixed undefined refs (`firstPlace`, `firstPlacementRelax`, `oppId`, `oppCorners`, `best`) that could halt turns.
- Fixed cases where AI removed a Straight during RS/RE/RT without re-placing/rotating.
- Fixed forced-play counters not resetting in some flows (draw, bottom, waive).
- Fixed deck-full state where drawing should remain allowed if **any** rotation cards remain in the deck.

### Bug Fixes (Oct 12 2025)

- Skipped-count reset fixed: turn-skip and elbow-skip counters no longer reset incorrectly at the start of a new turn.
- Forced-state restoration: Undo/Redo during a forced turn now fully restores the forced-placement lock so Bottom and End Turn remain disabled until you place or waive the card.
- AI legality fix: AI can no longer place tracks that only touch opponent tiles without connecting; placement now requires at least one friendly connection or a joined network.
- Adjacency highlight fix: human placement preview no longer shows island cells (no attachments) as valid except for the very first corner placement.
- Rotation spam: AI die-rotation frequency reduced—only rolls when a rotation could open progress.
- Undo safety: pending AI timers are cleared on Undo/Redo to prevent duplicate actions.
- Skip-count clamp: counters capped at 3 / 3 (max) to prevent “4 / 3” displays.
- HUD consistency: skipped counters, forced-turn banner, and tooltips now update instantly across Undo/Redo and AI turns.

---

## Files

- `RotoRouter.html` — the whole game (UI + logic)
- `MainMenu.html` — setup screen (board size, players, AI flags, names)
- `RotoRouterHelp.html` — in-game help (HTML)
- `RotoRouter-Rules.md` — printable rules (Markdown)
- _(optional)_ JSON save files produced by the Save/Load UI

---

## Quick Start

1. Open **`MainMenu.html`** (or `RotoRouter.html` → click **Main Menu**).
2. Pick a **Board Size**: 7×7 or 9×9.
3. Mark 2–4 colors **Active** (toggle AI if desired, edit names).
4. Click **New Game**.
5. On your turn:
   - **Draw** a card (if you don’t already hold one).
   - **Play** the card:
     - Place a track in a legal empty cell _or_
     - Use **RS/RE/RT** to rotate a matching placed tile _or_
     - Use **RC** to replace any placed tile with a Cross
     - _(Dual-use)_ RS/RE/RT/RC may also be placed as their base track.
   - Optionally take **Token Action**:
     - **Place** a token on your home corner (if a track is there), or
     - **Move** one token along connected tracks to any reachable empty cell.
   - **End Turn**.

**Controls**

- Rotate while previewing a placement: **Q/E** or sidebar **⟲/⟳**.
- Confirm placement: click again or **Place/Confirm**.
- **Undo / Redo (Global)**: roll back/forward the entire table state.
- **Save/Load**: snapshot to JSON and restore later.

---

## Deck & Cards

**Tracks**

- Straight (─), Elbow (└), T (┴), Cross (┼)

**Rotation/Replacement (dual-use)**

- **RS** rotate Straight _or_ place Straight
- **RE** rotate Elbow _or_ place Elbow
- **RT** rotate T _or_ place T
- **RC** replace any tile with Cross _or_ place Cross

**Deck scaling**

- Per-player decks scale with player count to keep supply healthy.
- Minimum per player:
  - **7×7:** Straight 7, Elbow 6, T 3, Cross 1
  - **9×9:** Straight 9, Elbow 8, T 3, Cross 1
- RS/RE/RT/RC scale too. Decks are created at **New Game**.

---

## Legal Placement & Networks

- Place only into an **empty** cell.
- The cell must be **adjacent** (N/S/E/W) to at least one track that is **connected to your network**.
- For two touching tracks to connect, the sides must have **matching, reciprocal exits**.
- **Opponent adjacency:**
  - Your normal placements may be adjacent to your own network.
  - You **may not** place adjacent to an opponent’s disconnected track unless the new tile **connects into** that opponent network via reciprocal exits.
  - Once a connection exists, both players may extend anywhere **adjacent to that connected network** (connected or merely adjacent).

---

## Tokens & Scoring

- **Place** a token on your home corner if a track is present there.
- **Move** a token along connected paths to any reachable empty cell (one token action per turn).
- **Score** by ending on an opponent’s corner → remove that token from the board.
- First player to **3 scored corners** wins; afterwards they’re auto-skipped.

---

## Forced Play

- **3 Skipped Tracks:** If you Bottom/skip 3 cards in a row, your **next turn** is **Forced Placement**.  
  HUD shows **“Forced: Draw & Place”**; **Bottom** and **End Turn** are disabled until you place the drawn card (if any legal spot exists).  
  If the drawn card has **no legal placement**, the force is **waived** and you may Bottom it.
- **3 Skipped Elbows:** If you skip 3 Elbows, the **next Elbow you draw** must be placed immediately (same waiver rule if no legal spot).

Forced state survives Undo/Redo correctly; after undoing during a forced turn, **Bottom** remains disabled again until the placement is resolved.

---

## AI (Phase-2 behaviors in this build)

- **Token targeting:** prefers paths that reduce distance to the **last unreached corner**; lingers near **high-leverage joints** (e.g., [1,5], [1,6]) to exploit future die rotations before moving away.
- **Placement legality fix:** AI cannot place adjacent to your disconnected tracks; it must connect into an opponent network to build alongside it.
- **Card economy:** biases saving **RS/RE/RT/RC** for endgame connections; will Bottom non-progress tracks when not forced.
- **Stall behavior:** on repeated non-improvement, cycles a card and attempts a single, conservative die rotation; re-evaluates without spamming rotations.
- **Save/Load safe:** AI flags and per-turn memory are serialized in saves.

---

## Standings & UI

- **Standings panel** in the sidebar shows rank (1st, T-2nd, etc.), color, name, and scores for **active** players only.
- Corners table includes a **Place** column mirroring standings.
- On touch: tap a legal cell to enter rotation mode; rotate with ⟲/⟳; tap again to confirm.

---

## Save / Load

Use the **Save** button to download a JSON snapshot. **Load** restores exactly, including turn order, decks, forced states, and AI status.

---

## Known Notes

- The “RC adopts replaced color” idea is **not implemented** yet (tracked for a later phase).
- If a trapped Straight or sealed corner appears, the **Dead-Straight Fix** or **Corner Fix** rules (below) apply at the start of your turn.

---

## Advanced Rules (implemented)

- **Dead-Straight Fix:** at the start of your turn, you may convert one fully trapped Straight into a Cross (free).
- **Corner Fix:** if your home corner is sealed by perpendicular Straights, you must convert it to a Cross before other actions.

See **RotoRouter-Rules.md** for a printable ruleset and **RotoRouterHelp.html** for a formatted, in-app reference.

---

## Save/Load

Use the **Save Game / Load Game** buttons in the sidebar to export/import a JSON snapshot. AI state is included.

---

## Known limitations

- AI favors short, safe progress; it won’t search arbitrarily deep “sacrifices”. This is intentional to keep turns brisk.
- Rare layouts may still require a few extra rotations to connect a final corner; this is mitigated by the stall logic.

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

### 36) Active Player Selection

- In Main Menu, each color now has an **Active** checkbox in addition to AI toggle and name.
- Any combination of colors may be chosen; there is **no required order** (doesn’t have to start with Red).
- Game requires a **minimum of 2 active players** (at least one may be AI).
- Maximum of **4 total players**, with up to **3 AI**.
- Inactive players are:
  - **Skipped automatically** during turn rotation.
  - Shown as dimmed / greyed-out in the Corners table with score `—` and “(inactive)” tag.
  - Displayed faintly on the board corners.
- Active/inactive state is persisted in Save/Load snapshots.

### 37) Touch / Tablet Rotation Support

- Players can now rotate new placements without a keyboard.
- On touch devices, tapping a legal empty cell enters **rotation mode** (ghost remains visible).
- Use the new **⟲ CCW** and **⟳ CW** buttons to rotate the ghost track.
- Tap the same cell again **or** press the **Place** button (now labeled **Confirm**) to finalize the placement.
- While in this mode, the ghost no longer disappears when moving the pointer away from the board.
- This improvement ensures complete touch/mouse parity for placement rotation.

### 38) Rotation normalization & token connectivity

- **Fix:** Some tiles carried non-orthogonal angles (e.g., 179°, 269°) due to legacy states or mid-animation clicks. This caused pathfinding to misread openings, so valid connections weren’t highlighted for token moves.
- **What changed:**
  - All rotations are **snapped to 0/90/180/270** when computing openings, when **placing/rotating** a tile, when **loading** a save, and in the **ghost preview**.
  - `rotDir(...)`, `rotatedOpenings(...)`, and `rotatedOpeningsType(...)` now operate on snapped angles; placement/rotation commits also store snapped values.
- **Player-visible effect:** Token movement highlights are now correct in edge cases.  
  _Example:_ A **T** on your corner with the stem facing **down**, a **Cross** to the right, and a **Straight (vertical)** below now correctly shows the straight below as reachable when you click **Token Action**.
- **Tech note:** Introduced `snap90(deg)` and routed all rotation math through it; history/animation behavior is unchanged.

### 39) Load restores saved player names (and AI flag)

- **Fix:** Loading a saved game always reverted player names to defaults. The loader now restores `players[].name` from the save (and `players[].ai` when present).

### 40) Confirm button for RS/RE/RT rotations

- **Fix:** When using **RS**, **RE**, or **RT** to rotate an existing track, the **Place** button did not change to **“Confirm”**, forcing players to click the tile again to commit.
- **New behavior:**
  - During any **rotate-existing** phase (`RS/RE/RT`), the **Place** button now changes to **Confirm**.
  - Pressing **Confirm** immediately finalizes the rotation — no extra tile click required.
  - Works identically to the **rotate-new** flow used during normal placement or tablet rotation mode.
- **UI text update:** The status line now reads:  
  _“Rotating (type): use ⟲/⟳ or Q/E to rotate; click again or press ‘Confirm’ to apply.”_  
  for consistent touch and keyboard guidance.

### 41) Standings Display and Dynamic Player Rankings

- Added a **Standings panel** at the top of the sidebar showing the **current ranking** of all **active players**.
- The display includes each player’s **standing** (1st, 2nd, 3rd, 4th), **color swatch**, **name**, and **score**, all on a single line.
- **Inactive players** are **not shown** in this panel.
- **Before the first scoring event**, standings remain hidden (no ranks shown).  
  Once any player reaches an opponent’s corner:
  - That player is labeled **1st** (bold).
  - All other active players are labeled **2nd**.
- After subsequent scores, standings automatically update and display ties as **T-1st**, **T-2nd**, etc.
- In the **Corners table**, a new **Place** column shows each player’s rank, using the same tie and bold-leader logic.
- The rank column and top standings remain blank until the first token scores.
- Visual cleanup:
  - Standings entries now use `white-space: nowrap` to prevent line breaks between color, name, and score.
  - Removed trailing separators (no “·” symbols).

### 42) Deck scaling with active players

- RotoRouter now scales each player’s personal draw deck based on how many players are active:

- Fewer players → more cards per player.
- Minimum connectivity guarantees ensure enough Straights/Elbows/Ts/Crosses to reach opponents, even with 2–3 players.

**Minimums (per player):**

- 7×7: Straight 7, Elbow 6, T 3, Cross 1
- 9×9: Straight 9, Elbow 8, T 3, Cross 1

- Rotation/Replacement (RS/RE/RT/RC) scale with the deck too. Decks are created at **New Game** using the selected board size and active player count.

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
