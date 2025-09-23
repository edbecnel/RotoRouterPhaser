
# RotoRouter — No-Block Edition

**Built:** 2025-09-23

This build removes **Block** tracks and adds a set of rules and UX safeguards to keep games flowing without soft-locks.

---

## Highlights
- **Forced Draw** when general track skips reach **3/3**.
- **Elbows Skipped (3/3)** → the next Elbow drawn **must be placed** that turn.
- **Corner access fairness**
  - **Non-blocking corners:** a token on its **owner’s** corner does **not** prevent an opponent from scoring there; the owner’s token remains.
  - **Sealed corner auto-fix:** a corner **Straight** jammed by **two perpendicular Straights** is auto-flagged; on the owner’s **next turn** the UI enters **Corner Fix** and the corner is replaced **for free** with a **Cross** (owned by the corner’s player).
- **RS/RE quality-of-life**
  - Confirm-first rotation on existing Straights/Elbows.
  - **0° preview persists** (no more “0 is falsy” discard).
  - **Selected tile highlight** while rotating an existing tile (bold dashed outline + ghost preview).
- **Token Action safety:** clicking **Token Action** with **no tokens** (and no legal corner placement) no longer auto-advances the turn; a status + toast explains instead.
- **HUD polish:** explicit red warnings and longer-lived toasts for important states.

---

## What’s new (2025-09-23)

### 1) Corner Fix warning + toast duration
- When a seal is detected, we toast: *“<Color> corner sealed — corner will be auto-fixed on their next turn.”*  
  Toasts now support custom durations and the seal toast shows ~**4.5s**.
- On the owner’s next turn, **Corner Fix** starts automatically and locks actions with a red warning:  
  **“Your corner is sealed. Click on your corner track to replace it with a cross track.”**
- Clicking the corner replaces it with a **Cross** (free), then normal play resumes.

### 2) RS/RE rotate-existing clarity
- When you click an existing **Straight/Elbow** with **RS/RE** in hand, the chosen cell becomes **visibly active** (highlight + dashed outline), and the **ghost preview** rotates with **Q/E**.
- Confirming on the same cell commits the preview **before** occupancy checks.
- Rotation to **0°** now **sticks** via the `??` fix (no more fallback to the old rotation).

### 3) Token Action guard
- If there are no tokens to act on and your corner isn’t placeable, we **do not** auto-advance.  
  Instead we show: *“No tokens to apply an action to. Place a token at your corner first or Draw/Place tracks.”*  
  The action remains available for later.

---

## Rules in detail

### Forced Draw (3/3 track skips)
- If a player’s general `Skipped` count reaches **3**, **Draw** becomes **Ready (Forced)** while **End Turn** is disabled (Override hidden unless a card is in hand).  
- After a placement, skip resets to **0**.

### Elbows Skipped (3/3) → next Elbow must be placed
- Each bottomed Elbow or turn ended while holding an Elbow increments `Elbows Skipped`.
- At **3/3**, the next **Elbow** drawn must be **placed** that turn (Bottom/End Turn disabled).  
- Placing an Elbow resets the elbow counter.

### Non-blocking corners
- Moving onto an opponent’s corner that already has its owner’s token **scores** for the mover and **does not remove** the owner’s token.

### Sealed corner → Corner Fix (forced Cross)
- **Detector:** fires **only** for the **perpendicular Straights jam**: corner tile is **Straight**, and both adjacent tiles are **Straight** and **perpendicular** to the corner’s current orientation.
- **Timing:** detection runs on every topology change (place/RS-RE/RC/apply). Corner Fix runs at the **start of the owner’s next turn**.
- **UI:** red warning; **Draw** shows **“Locked (Corner Fix)”**; all actions disabled until the player clicks their own corner; we then replace it with a **Cross** (free).

---

## HUD elements to watch
- **Skipped** and **Elbows Skipped** tags highlight in red when thresholds hit (with descriptive suffixes).
- **Draw/Place/Bottom** enable/disable rules follow the forced-draw and elbow-forced states.
- **Phase** shows `cornerFix` during the fix subphase.

---

## Recommended deck composition (per player)

**Updated:** 2025-09-23

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
- Faster jam-breaking (9×9): +1 **Cross** (keep **RCross** rare).
- Shorter 7×7: **Straight 9**, **Elbow 5** (total 22).

### Optional: auto-sized deck
Use a size-aware `makeDeck(size)` and pass `state.N` when (re)building player decks.

```js
function makeDeck(size){
  const cfgMap = {
    7: { Straight:10, Elbow:6, RStraight:3, RElbow:2, Cross:2, RCross:1 },
    9: { Straight:13, Elbow:8, RStraight:4, RElbow:3, Cross:2, RCross:2 },
  };
  const cfg = cfgMap[size] || (size <= 7 ? cfgMap[7] : cfgMap[9]);
  return {
    draw: shuffle([
      ...Array(cfg.Straight).fill(TrackCard.Straight),
      ...Array(cfg.Elbow).fill(TrackCard.Elbow),
      ...Array(cfg.RStraight).fill(TrackCard.RStraight),
      ...Array(cfg.RElbow).fill(TrackCard.RElbow),
      ...Array(cfg.Cross).fill(TrackCard.Cross),
      ...Array(cfg.RCross).fill(TrackCard.RCross),
    ]),
    discard: []
  };
}
```

---

## Changelog
- **2025-09-23**
  - Corner Fix: red warning & longer toast; explicit perpendicular-straights detector.
  - RS/RE: selected-cell highlight and 0° confirm fix.
  - Token Action: no auto-advance when no tokens to act on.
  - README: consolidated rules, deck recommendations, and UX notes.
- **2025-09-20** HUD Skipped counter with visual red warning.
- **2025-09-16** RS/RE rotation confirm-first fix.

---

## Run
Open **`RotoRouter.html`** in a modern desktop browser. Hard-refresh (Ctrl/Cmd+Shift+R) after swapping files.
