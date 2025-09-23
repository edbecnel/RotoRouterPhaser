
# RotoRouter — No‑Block Edition

**Build:** 2025-09-23

This build removes **Block** tracks and adds rules + UX to prevent soft‑locks while keeping games fast and readable.

---

## Highlights
- **Forced Draw** when *general* track skips reach **3/3** (Draw becomes **Ready (Forced)**; End Turn is disabled until a placement or Override when a card is in hand).
- **Elbows Skipped (3/3)** → the next **Elbow** drawn **must be placed** that turn (Bottom/End Turn disabled).
- **Corner access fairness**  
  - **Non‑blocking corners:** A token resting on its **owner’s** corner does **not** block opponents from scoring there; the owner’s token remains.
  - **Sealed corner auto‑fix:** A corner **Straight** jammed by **two perpendicular Straights** is auto‑flagged. On the owner’s **next turn** the UI enters **Corner Fix** and the corner is replaced **for free** with a **Cross** (owned by the corner’s player). Clear red warning shown until the fix is applied.
- **RS/RE quality‑of‑life**  
  - Click an existing **Straight/Elbow** with **RS/RE** in hand to enter **rotate‑existing** (tile is highlighted; ghost rotates with **Q/E**).  
  - **Confirm‑click** commits the preview angle **without re‑animating** (no “snap back then spin”).
- **RC improvements**  
  - RC can **replace any existing track**.  
  - RC can be **placed as a new Cross** on your **own empty corner**.  
  - If your corner already has a track, RC can also be placed on any **empty orthogonal neighbor** of your corner.
- **Animated ghosts**  
  - Track ghost follows the cursor in **Place** and **tweens** smoothly on **Q/E** (no snap).  
  - Token ghost appears during **Token** actions; both ghosts show a **dashed red box** on illegal cells.
- **Visible rotations**  
  - RS/RE preview rotates smoothly; board‑wide **Apply** uses slower, clearly visible tweens.
- **Token Action safety**  
  - Clicking **Token Action** with no legal target no longer advances the turn; a status + toast explains what’s missing.
- **HUD polish**  
  - Important warnings use red emphasis; the corner‑seal toast remains visible longer.  
  - Helper line reads: “**Hover shows a ghost. Click to place. Rotate with Q/E.**” on a single line.

---

## Rules in Detail

### 1) Forced Draw (3/3 general track skips)
- If a player’s general `Skipped` counter hits **3**, **Draw** shows **Ready (Forced)**, **End Turn** is disabled (Override only when a card is in hand), and play cannot proceed until the player **Draws and Places**.  
- After any placement, `Skipped` resets to **0**.

### 2) Elbows Skipped (3/3) → next Elbow must be placed
- Bottoming an Elbow or ending the turn while holding one increments **Elbows Skipped**.
- At **3/3**, the next **Elbow** drawn **must** be placed that turn. Bottom/End Turn are disabled until the Elbow is placed. Placing an Elbow resets the elbow counter.

### 3) Corner access fairness
- Moving a token onto an opponent’s corner **scores** for the mover even if the owner’s token already occupies that corner; the owner’s token is **not** removed.

### 4) Sealed corner → Corner Fix (forced Cross)
- **Detector:** corner tile is **Straight**, both adjacent tiles are **Straight** and **perpendicular** to the corner’s current orientation (explicit “perpendicular straights jam”).  
- **Timing:** detection runs on each topology change (place/RS‑RE/RC/apply). On the **owner’s next turn**, the UI switches to **Corner Fix**: all actions are locked; a red warning instructs the player to click their corner; clicking replaces the corner with a **Cross** (free); play resumes.

### 5) Token movement (BFS over pipes)
- Tokens move along **reciprocally connected pipes**; **ownership does not matter** once networks connect.
- Tokens may **traverse through** occupied cells but may **not stop** on them.  
  - Exception: an opponent’s corner with **their** token is a **valid scoring terminal**.
- If a selected token has **no reachable destinations**, we keep you in Token mode, re‑highlight only movable tokens (plus corner placement if legal), and show *“That token is stuck (no connected destinations). Choose a different token or click End Turn.”*

---

## Controls & UI

- **Draw / Place / Bottom** as usual.  
- **Rotate (Q/E)** while hovering an empty cell in Place to rotate the **ghost**; animation ~**400 ms**.  
- **RS/RE:** click an existing Straight/Elbow to select; rotate with Q/E; **click again to confirm**.  
- **RC:** click any track to replace with a Cross, or click your **own empty corner** (or its orthogonal neighbors if your corner has a track) to place a new Cross.  
- **Roll Die / Apply:** meshed rotation; neighbors alternate directions. Rotation tween ~**650 ms**.  
- **Show connection edges** checkbox helps visualize pipe connectivity.
- Sidebar helper: **“Hover shows a ghost. Click to place. Rotate with Q/E.”**

---

## Recommended Deck Composition (per player)

Balanced starting counts that scale with board size and current rules.

### 9×9 board
- Straight: **13**  
- Elbow: **8**  
- RStraight: **4**  
- RElbow: **3**  
- Cross: **2**  
- RCross: **2**  
**Total:** 32 cards

### 7×7 board
- Straight: **10**  
- Elbow: **6**  
- RStraight: **3**  
- RElbow: **2**  
- Cross: **2**  
- RCross: **1**  
**Total:** 24 cards

**Tuning knobs**
- More flexibility: +1 **RStraight** and +1 **RElbow**.  
- Faster jam‑breaking (9×9): +1 **Cross** (keep **RCross** rare).  
- Shorter 7×7: **Straight 9**, **Elbow 5** (total 22).

**Optional: auto‑sized deck**

```js
function makeDeck(size){
  const cfgMap = {{
    7: {{ Straight:10, Elbow:6, RStraight:3, RElbow:2, Cross:2, RCross:1 }},
    9: {{ Straight:13, Elbow:8, RStraight:4, RElbow:3, Cross:2, RCross:2 }},
  }};
  const cfg = cfgMap[size] || (size <= 7 ? cfgMap[7] : cfgMap[9]);
  return {{ /* build deck from cfg */ }};
}
```

---

## Changelog
- **2025-09-23**
  - Animated **track ghost** on Q/E; always‑visible drag ghost with illegal dashed box.
  - **Token ghost** during Token actions; dashed illegal indication.
  - **RS/RE**: confirm‑click commits preview angle (no extra spin); highlighted selection.
  - **RC**: may place Cross on **own empty corner** and on **empty orthogonal neighbors** if the corner already has a track.
  - Token movement uses **BFS over pipes**, ignoring ownership; pass‑through over tokens; can stop on opponent’s own corner for scoring.
  - Corner‑fix warning + longer‑lived toast.
  - Helper hint forced to a **single line** (no stray space before the period).
- **2025‑09‑23** Consolidated rules, deck recommendations, and HUD notes.
- Earlier: Force Draw & Elbow‑force; Non‑blocking corners.

---

## Run
Open **`RotoRouter.html`** in a modern desktop browser. Hard‑refresh (Ctrl/Cmd+Shift+R) after swapping files.
