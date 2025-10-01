# RotoRouter — Quick How-To

## Objective

Be the first to **score 3 corners** by moving your tokens onto _three different opponents’ corner squares_.

- Each player has **3 tokens**.
- When a token reaches an opponent’s corner, it **scores** and is **removed from the board**.
- When a player reaches **3/3**, they’re finished and are auto-skipped.

---

## Setup

- Choose **board size**: _7×7_ or _9×9_ (Main Menu).
- Each player picks a **color** and **name** (AI flags optional).
- Everyone starts with **no tokens on the board**. Your **home corner** is your color’s corner.

---

## The Deck

### Track cards (place new tracks into empty cells)

- **Straight (─)** — 2 exits (E↔W)
- **Elbow (└)** — 2 exits (turn; base N↔E)
- **T (┴)** — 3 exits (base N, E, W; closed S)
- **Cross (┼)** — 4 exits (N, S, E, W)

### Rotation & Replacement cards (dual-use)

> These cards can either **act on existing tiles** _or_ be **used to place new tracks** just like their normal counterparts.

- **RS** — rotate an existing **Straight** _or_ place a new **Straight**
- **RE** — rotate an existing **Elbow** _or_ place a new **Elbow**
- **RT** — rotate an existing **T** _or_ place a new **T**
- **RC** — replace _any_ existing tile with a **Cross** _or_ place a new **Cross**

_Tip:_ while a placement “ghost” is shown, use **Q/E** to rotate before confirming.

---

## Turn Flow

1. **Draw** a card (if you don’t already hold one).
2. **Play a card**:
   - Place a track (Straight/Elbow/T/Cross) in a legal empty cell, **or**
   - Use **RS/RE/RT** to rotate a matching placed tile, **or**
   - Use **RC** to replace any placed tile with a Cross,
   - _(Dual-use)_ or play RS/RE/RT/RC as their **normal track** into an empty cell.
3. **Token Action** (optional):
   - **Place** a new token on your **home corner** (if empty & connected), **or**
   - **Move** one of your tokens along connected tracks to any reachable empty cell.
4. **End Turn**.

## Forced Play Rules

- **3 Skipped Tracks**: If you skip 3 track cards in a row, on your **next turn** you must **Draw and Place** the card drawn (if legal). Bottom and End Turn are disabled until you place it.
- **3 Skipped Elbows**: If you skip 3 Elbows, the **next Elbow drawn must be placed** immediately (if legal). Bottom and End Turn are disabled until the Elbow is placed.
- If a forced card has **no legal placement**, the force is **waived** — you may Bottom it and end your turn normally.

---

## Legal Placement & Connections

**Track placement**

- The cell must be **empty**.
- It must be **adjacent** (N/S/E/W) to at least one track **connected to your network**.
- All touching sides must have **matching, reciprocal exits**.

**Connected network**

- Tracks connect through continuous chains of matching exits.
- Empty cells adjacent to that network may be valid placements (the game highlights them).

**Rotation / replacement**

- **RS/RE/RT**: click a matching placed tile → rotate with **Q/E** → click again to confirm.
- **RC**: click any placed tile to turn it into a **Cross** (rotate before confirming if desired).
- _(Dual-use)_ All rotation cards can also be **placed as new tracks** of their base type.

---

## Tokens & Scoring

- **Place** a token: onto your **home corner** if it’s empty and connected.
- **Move** a token: along connected paths to any reachable empty cell (one move per turn).
- **Score**: end a move on an **opponent’s corner** → score that corner; the moving token is **removed from the board**.
- **Finish**: upon **3/3** scored corners, your turn ends immediately and you are auto-skipped thereafter.

---

## Bottoming (Recycle)

**Bottoming** means moving the held card to the **bottom of your personal draw pile** (it is **recycled**, not discarded).

- If the held card has **no legal placement**, you may **Bottom** it **with no penalty**.
- If the card **does** have a legal placement and you Bottom it anyway, it counts as a **skip** (shown in HUD).
- A bottomed card **returns later** when it cycles back up.

---

## Dead-Straight Fix

If a Straight track becomes completely trapped by perpendicular Straights (cannot extend):

- At the **start of your turn**, such cells are highlighted.
- You may replace **one** trapped Straight with a **Cross** for free.
- Tokens on that cell remain in place.

---

## Corner Fix

If your own home corner is sealed by perpendicular Straights:

- At the **start of your turn**, the corner is highlighted.
- You must replace it with a **Cross** before taking other actions.
- A red HUD warning appears until the fix is made.

---

## Ending the Game

- When all remaining players are finished/out, the game ends and shows:  
  **All players have finished — game over.**
- **Winner:** first to reach **3/3** scored corners. If multiple finish in one round, earliest to 3/3 wins.

---

## Edge Cases

- A “forced” card with **zero legal placements** is **waived** (prevents deadlocks).
- HUD shows the **actual** token count on-board + removed; if a legacy Undo state causes a mismatch, a ⚠ appears, while illegal extra placements remain blocked.
