# Crossnumbers — Reverse-Engineered Game Specification

**Target implementation:** Unity 2022 LTS or newer, WebGL-capable  
**Purpose:** Source-of-truth gameplay/UI specification for another AI coding agent.  
**Scope:** Recreate the supplied Crossnumbers game behavior from the screenshots, the supplied master instructions, and the player's direct observations. No arithmetic mechanic exists; the puzzle is based on positional digit matching at crossword-style intersections.

---

## 1. Evidence and priority rules

This document uses three confidence classes:

- **Observed** — directly visible in a supplied screenshot, explicitly stated in the master instructions, or directly reported by the player from hands-on play.
- **Inferred** — strongly implied by the observed behavior but not directly demonstrated in the supplied evidence.
- **Unknown** — cannot be established accurately from the available evidence. Do not invent these behaviors without a deliberate design decision.

### Evidence priority

When sources differ in precision, use this order:

1. **Player's direct game-experience observations** — highest priority for runtime behavior.
2. **Screenshots** — highest priority for visible UI, layout, colors, states, and demonstrated puzzle data.
3. **Master instructions (`original manual.txt`)** — authoritative for case-study deliverables and feature requirements, but not evidence of an additional arithmetic mechanic.
4. **Inference** — only where needed to connect observed states.


For implementation-level visual tokens (measured/estimated colors, line weights, typography, spacing, and responsive sizing), use `VISUAL_DESIGN_SPEC.md` as the visual source of truth. Screenshot-observed behavior in this file still takes precedence if a conflict is found.

### Supplied screenshots

| Evidence ID | File | Main evidence |
|---|---|---|
| `IMG-MAIN` | `main_screen.jpg` | Main screen, logo, Settings entry, New Game button. |
| `IMG-SETTINGS` | `settings.jpg` | Sound/Vibration settings, Help, Privacy Policy, Done. |
| `IMG-L1-START` | `1st_level_start.jpg` | Tutorial level start, 5×5 topology, two 5-digit pieces. |
| `IMG-L1-PLACED` | `1st_level_1st_item_placed.jpg` | Player-placed number rendered on white cells while game-supplied cells remain blue. |
| `IMG-L1-HOVER-H` | `1st_level_horizontal_placement.jpg` | Horizontal targeting/placement feedback. |
| `IMG-L1-HOVER-V` | `1st_level_vertical_placement.jpg` | Automatic vertical orientation and target feedback. |
| `IMG-L2-DRAG` | `2nd_level_different_digit_items.jpg` | Larger board, multiple digit-length groups, toolbar, selected-piece feedback. |
| `IMG-MISPLACE` | `misplacement.jpg` | Contradicting intersection feedback in red/pink. |
| `IMG-WIN` | `win_screen.jpg` | Completed grid, Congratulations, confetti, New Game, Main. |
| `MANUAL` | `original manual.txt` | Unity target, required core mechanics, ≥5 levels, Monthly Pass, WebGL/documentation deliverables. |

The supplied portrait screenshots are 945×2048 reference images.

---

# 2. Core game concept

## 2.1 Puzzle premise

**Observed**

Crossnumbers is a crossword-like number-placement puzzle. The player is given a grid containing active cells and blocked cells plus a bank of multi-digit number pieces. Number pieces contain **3 to 7 digits** in the normal game and are grouped below the puzzle by digit count.

The player places an entire number into a horizontal or vertical slot having the same number of cells. Where horizontal and vertical numbers cross, the digit contributed by both numbers must be the same, analogous to letters matching in a crossword or constraints matching in Sudoku.

There is **no arithmetic mechanic**. Numbers are digit strings, not values used in calculations. No addition, subtraction, multiplication, division, equality expressions, or arithmetic equations are part of the original mechanic represented here.

## 2.2 Meaning of “equation balancing” in the case-study brief

**Observed + clarified by player**

For this recreation, the brief's “equation balancing” requirement should be implemented as the game's visible constraint behavior: **intersection digits must agree**. Do not add arithmetic operators or arithmetic equation solving.

## 2.3 No failure state

**Observed**

There is no level-fail condition, life system, mistake counter, or loss state. The player may make wrong placements, rearrange pieces, use hints, remove pieces, or restart the level. Progress continues until the puzzle is solved or the player leaves/restarts.

---

# 3. Terminology

- **Cell** — one square of the board grid.
- **Blocked cell** — dark navy cell that can never contain a digit.
- **Active cell** — a non-blocked grid cell belonging to at least one playable number slot.
- **Game-placed / locked cell** — an active cell whose digit was supplied by the game. It has a light-blue background and cannot be picked up or removed by the player.
- **Player-filled cell** — an active cell filled as part of a number placed by the player. Its normal background is white.
- **Slot / space** — a contiguous horizontal or vertical sequence of active cells intended to hold one number piece of exactly the same digit count.
- **Piece / number** — one draggable multi-digit string from the number bank.
- **Intersection** — one active cell shared by a horizontal slot and a vertical slot.
- **Candidate slot** — any slot whose length equals the selected piece's digit count. Candidate slots glow orange while the piece is selected.
- **Target slot** — the candidate slot currently being hovered/targeted by the selected piece.
- **Contradiction / mismatch** — a crossing position where the selected/placed piece supplies a different digit from the number already crossing that position.
- **Eviction** — automatic removal of an already player-placed number because a newly placed number contradicts it at an intersection, or because another number replaces it in the same slot.
- **Bank** — the grouped number list below the grid.
- **Digit class** — one bank group defined by number length, e.g. `3 digits`, `6 digits`, or `7 digits`.
- **Digit-class focus** — a non-placement selection state created by pressing a digit-class card or board space. It highlights every slot of that length and the corresponding bank group without selecting a number piece.

---

# 4. Visual ownership rule: blue vs white cells

## 4.1 Light-blue cells

**Observed**

A light-blue cell means **the digit was put there by the game**, not by the player.

Properties:

- The digit is locked.
- It cannot be picked up.
- It cannot be removed by normal player interaction.
- It acts as a persistent constraint for any number using that cell.
- Initial puzzle givens use this state.
- A digit inserted by the Hint system is also game-supplied and should therefore use this locked light-blue state.

## 4.2 White cells

**Observed**

Digits belonging to a number placed by the player render on white cells in the settled board state.

A player's whole placed number remains movable/removable even though its individual cells display normally on white backgrounds.

## 4.3 Shared cells

**Observed / inferred consequence**

If a player's number passes through an already locked light-blue digit and the value agrees, the cell remains visually light blue because ownership of that cell belongs to the game, not the player.

If two player-placed numbers agree at an intersection, the shared cell is a normal white player-filled cell unless it was previously locked by the game.

---

# 5. Screen inventory

## 5.1 Main screen

### Observed

- Very pale blue/white background.
- Large faint rotated crossword-number pattern as decorative background art.
- Centered `crossnumbers` logo in dark navy.
- Gear icon in the upper-right area.
- Large bright-blue pill-shaped **New Game** button near the bottom.
- No visible level-select screen, score, currency, timer, profile, stars, lives, or streaks.

### Interactions

- Gear → Settings.
- New Game → starts/enters the player's current progression level.

### Progress-dependent behavior

**Observed at behavior level**

If the player left an unfinished level and later returns, the saved in-level progress exists and the player can either:

- **Continue** the saved level state, or
- **Restart** that level from its initial state.

**Unknown**

The exact UI used to present the Continue-vs-Restart choice is not shown in the supplied screenshots.

---

## 5.2 Settings screen

### Observed

- White top bar.
- Centered title **Settings**.
- **Done** action at upper right in blue.
- Light gray background.
- Rounded white group containing:
  - **Sound** toggle, shown ON.
  - divider.
  - **Vibration** toggle, shown ON.
- Separate rounded rows for:
  - **Help** + right chevron.
  - **Privacy Policy** + right chevron.
- Enabled toggles use a blue track and a white thumb on the right.

### Interactions

- Sound toggles game sound preference.
- Vibration toggles haptic preference.
- Help opens Help content.
- Privacy Policy opens Privacy Policy content/destination.
- Done exits Settings to the prior screen.

### Unknown

- Exact Help content.
- Exact Privacy Policy destination/content.
- Exact list of sound and vibration events.

---

## 5.3 Tutorial / first level gameplay screen

### Observed

- First level is a tutorial.
- 5×5 board.
- One horizontal slot and one vertical slot crossing each other.
- One number group labeled **5 digits**.
- Two pieces: `12345` and `54321`.
- Two game-supplied light-blue locked digits.
- No later-level top toolbar is visible in the supplied tutorial screenshots.

### Tutorial purpose

**Observed from player description + screenshots**

The level teaches the two basic placements:

1. one horizontal number placement;
2. one vertical number placement.

This introduces same-length candidate highlighting, automatic orientation, intersection matching, and the visual difference between game-supplied blue digits and player-supplied white digits.

---

## 5.4 Standard gameplay screen

### Observed

Later levels show:

- Back button at top left in a circular control.
- Restart button next to Back, using a clockwise-arrow icon.
- Hint/light-bulb button at top right.
- Small blue badge on the Hint control displaying `5` in the captured state.
- Board centered below the toolbar.
- Number groups below the board.
- Digit-length categories observed across the game: 3, 4, 5, 6, and 7 digits.

### Interactions

- Back leaves the current gameplay screen.
- Restart restarts the current level.
- Hint fills one random currently empty active digit cell with the correct solution digit.
- Number pieces can be selected from the bank or picked back up from the board.

### Unknown

- Exact Back destination and whether it asks for confirmation.
- Exact Restart confirmation UI, if any.
- Whether the visible `5` hint count is per level, global, or otherwise scoped.

---

## 5.5 Win screen

### Observed

- Full-screen saturated blue background.
- Large yellow/orange **Congratulations!** heading.
- Multicolored confetti.
- Completed board displayed below the heading.
- Large white pill **New Game** button.
- **Main** text action below it.
- Completed grid is presented cleanly without active hover/conflict colors.

### Progression behavior

**Observed / strongly implied by non-replayable progression**

- Completing a level advances progression.
- Previous completed levels cannot be replayed.
- **New Game** after a win proceeds into the next available progression level rather than a completed earlier level.
- **Main** returns to Main.

### Unknown

- Behavior after the final authored level.
- Exact win transition duration and confetti animation timing.

---

# 6. Top-level application state machine

```text
AppStart
  -> MainMenu

MainMenu
  -> Settings                         [gear]
  -> ResumeChoice / GameplayLoading   [New Game, depending on saved unfinished level]

Settings
  -> MainMenu or previous screen      [Done]
  -> Help                             [Help]
  -> PrivacyPolicy                    [Privacy Policy]

ResumeChoice
  -> Gameplay                         [Continue saved state]
  -> Gameplay                         [Restart current level from initial state]

Gameplay
  -> Gameplay                         [normal placement/rearrangement]
  -> Gameplay                         [Hint]
  -> Gameplay                         [Restart]
  -> MainMenu/previous screen         [Back; exact destination unknown]
  -> WinCelebration                   [completion condition satisfied]

WinCelebration
  -> GameplayLoading(next level)      [New Game]
  -> MainMenu                         [Main]
```

### Persistence rule

**Observed**

The current in-progress level state is saved between sessions. When the user returns, they can continue that exact progress or restart the current level.

At minimum, persistence must preserve all gameplay-affecting state needed to reconstruct the board accurately, including player placements and game-placed hint cells.

---

# 7. Gameplay state machine

The core interaction is **rearrangement**, not pass/fail validation. Red feedback warns of a contradiction, but the player can still place the new number; doing so resolves the conflict by removing the contradicting player number.

## 7.1 `Gameplay.Idle`

### Observed

- Unplaced pieces are visible in their length-group bank cards.
- Player-placed pieces are visible in the board and may be picked up again.
- Game-placed digits are light blue and locked.
- Player-filled digits are white.
- No candidate slots are highlighted when no number or digit class is selected.

### Transitions

- Select/press an unplaced bank number → `PieceSelected`.
- Press a digit-class card/header such as `6 digits` → `DigitClassFocused`.
- Press a board space/cell without beginning a piece drag → `DigitClassFocused`.
- Begin dragging a player-placed board number → `PieceSelectedFromBoard`.
- Hint → `ApplyHint`.
- Restart → reset level.

---


## 7.2 `DigitClassFocused`

### Observed from player description

The player can inspect slot classes without selecting a specific number.

#### Pressing a digit-class card

Pressing a bank group/card such as **`6 digits`**:

1. highlights the entire `6 digits` class card/table in orange;
2. highlights **every 6-cell slot** on the board in orange;
3. does **not** select or move a number piece;
4. does **not** perform correctness validation;
5. does **not** create green/red placement feedback.

The same rule applies to every digit class present in the level.

#### Pressing a board space

Pressing a cell belonging to a non-crossing slot:

1. resolves the slot containing that cell;
2. reads that slot's length `x`;
3. highlights every `x`-digit slot in orange;
4. highlights the corresponding `x digits` bank class/card in orange.

The board and bank therefore mirror one another: choosing a length class highlights all spaces of that length, and choosing a space highlights its length class.

### Crossing-cell priority and cycling

A cell shared by a vertical and horizontal slot is ambiguous. The original behavior is explicitly:

1. **First press on that crossing cell:** choose the **vertical** slot.
2. Highlight every slot with the vertical slot's digit length and highlight that digit-class card.
3. **Second consecutive press on the same crossing cell:** switch to the **horizontal** slot.
4. Highlight every slot with the horizontal slot's digit length and highlight that digit-class card.
5. Further consecutive presses on that same crossing cell alternate vertical → horizontal → vertical → horizontal.

If the vertical and horizontal slots happen to have the same length, the visible length-class highlight may look unchanged, but the internally focused slot/orientation still alternates.

Pressing a different crossing cell resets the cycle: that new cell again starts with its vertical slot.

### Interaction with piece selection

Digit-class focus is informational only. Selecting or beginning to drag a number piece supersedes it and enters the normal piece-selection/placement state. The selected piece then uses the existing candidate-slot, green-valid, red-conflict, and replacement rules.

A short tap on a player-filled cell may be interpreted as slot/class inspection, while a drag beginning from a player-placed number is used for moving that number. This tap-versus-drag split is the recommended implementation disambiguation; the gameplay requirement is that both class inspection and player-piece movement remain available.

### Persistence

Digit-class focus is transient UI state and does **not** need to be saved. Reloading/returning to a level may begin with no class focus.

---

## 7.3 `PieceSelected`

### Observed

When a number is pressed/selected:

- The game automatically handles its orientation; the player does not manually rotate pieces.
- Every slot with the **same number of cells** becomes highlighted orange.
- Candidate highlighting is based on slot length, not on whether the number is ultimately correct for that slot.
- Same-length candidate slots may include empty or currently occupied slots.
- The selected number's bank/category receives selected-state feedback consistent with the screenshots.

### Automatic orientation

**Observed**

The number automatically rotates/alignment-adjusts to horizontal or vertical according to the available/targeted slot orientation. Horizontal slots display digits left-to-right; vertical slots display digits top-to-bottom.

The implementation must not require a manual rotate button or rotate gesture.

### Transition

- Hover/drag over a same-length target → one of the target feedback substates below.
- Release/remove without choosing a board target → piece returns to/remains available in the bank, subject to the pickup-removal gesture details described later.

---

## 7.4 `HoverMatchingEmptyOrCompatibleSlot`

### Condition

The target slot:

- has the same cell count as the selected number;
- is not simply being replaced by another number in that exact slot; and
- has no contradictory crossing player number or locked digit.

### Observed feedback

- The target space glows green.
- The selected number glows green.
- The selected number aligns to the target's exact cell positions and orientation.

### Release behavior

- Placement is committed.
- The new number becomes a player-placed movable piece on white cells, except any pre-existing game-placed light-blue cells retain blue styling.
- Matching crossing player numbers remain in place.

---

## 7.5 `HoverContradictingSlot`

### Condition

The selected number fits the slot length but would place at least one digit that disagrees with a digit supplied at an intersection by another already player-placed number.

### Observed feedback

- The contradicting digit is highlighted red/pink.
- The conflicting space/number is shown with red/pink contradiction feedback.
- The screenshot demonstrates the already placed crossing number/strip glowing red/pink.
- Other same-length candidates remain orange.

### Critical observed rule: contradiction does not block placement

The player may place the selected number **anyway**.

On commit:

1. The newly placed number remains in the target slot.
2. Any already player-placed crossing number that contradicts the new number is automatically removed from its slot.
3. The evicted number becomes available to place again.
4. The shared cell is now owned visually by the newly placed number unless it is a locked game-supplied cell.

Example supplied by the player:

- A new vertical number contradicts a digit in an existing horizontal number.
- The vertical number is placed anyway.
- The existing horizontal number is removed.

### Multiple simultaneous contradictions

**Inferred**

If one placement contradicts more than one player-placed crossing number, every contradicted crossing player number should be evicted so the board does not retain mutually inconsistent player pieces.

### Locked-blue contradiction

**Unknown**

Game-supplied light-blue cells cannot be removed. The exact behavior when a player attempts to place a same-length number that contradicts a locked blue digit is not established by the current evidence. This remains an implementation-critical open question.

---

## 7.6 `HoverOccupiedSameSlot`

### Condition

A selected number is hovered over a same-length slot already occupied by another player-placed number.

### Observed feedback

- The existing number and the incoming number glow orange.
- The state communicates replacement rather than a valid-green match.

### Release behavior

**Observed**

- The incoming number replaces the number already occupying that slot.
- The old number is removed from the slot and becomes available again.
- The incoming number becomes the slot's current player-placed number.

### Interaction with crossing contradictions

**Inferred**

After replacing the slot occupant, normal intersection contradiction resolution should run for the incoming piece: any other player-placed crossing pieces that contradict it are evicted.

The precise color-precedence when an occupied-slot replacement also creates a crossing contradiction is **Unknown** from screenshots.

---

## 7.7 `PieceSelectedFromBoard`

### Observed

Every player-placed number can be picked back up and removed from its space. Player placements are never permanently locked merely because they were successfully placed.

When the number is removed/moved:

- Its old slot becomes available again.
- The piece can be placed into another same-length slot.
- If it is fully removed from the board, it returns to the bank/available pool.
- Any crossing number that remains on the board retains its own digits.
- Locked light-blue cells remain untouched.

### Unknown

The exact removal gesture is not demonstrated precisely enough to distinguish among:

- dragging the piece back toward the bank;
- dragging it off any valid slot and releasing;
- tapping/picking it up and canceling placement;
- another equivalent gesture.

The implementation must support functional removal/repositioning, but the exact UX gesture should be confirmed if pixel/interaction parity is required.

---

## 7.8 `ApplyHint`

### Observed

Using a hint fills **one random empty digit cell**, not a whole number and not a whole slot.

Required behavior:

1. Choose one currently empty active board cell.
2. Fill it with the correct solution digit for that cell.
3. Mark it as game-supplied/locked.
4. Render it light blue.
5. The player cannot pick up or remove that hinted digit afterward.

A hinted intersection digit constrains both slots that use the cell.

### Inferred

- The bulb badge is a remaining-hint counter.
- Using a hint likely decrements it by one.

### Unknown

- Whether the random choice is uniform among all empty active cells.
- Whether the hint balance is per level, persistent/global, or replenished by another system.
- What happens when no empty active cell remains.

---

## 7.9 `Restart`

### Observed

The player can restart the level from the top toolbar. There is no failure requirement to restart.

Restart must return the current level to its authored initial state:

- remove all player-placed numbers;
- restore all pieces to their original availability;
- restore original game-supplied locked cells;
- remove hint-created locked cells that were not part of the authored initial board;
- clear transient selection/highlight states;
- overwrite the saved in-level progress with the fresh level state.

### Unknown

- Whether restarting refunds hint uses consumed during that attempt.
- Whether a confirmation dialog appears.

---

# 8. Grid and slot rules

## 8.1 Grid topology

**Observed**

- Board is a rectangular matrix of square cells.
- Supplied examples include 5×5, 6×6, and 8×8 boards.
- Dark navy cells are blocked/inactive.
- Non-blocked cells are active puzzle cells.
- Active runs form horizontal and vertical slots.
- Slots can intersect orthogonally.

## 8.2 Slot length

**Observed**

A number can target a slot only if the number of digits exactly equals the number of cells in the slot.

Normal number lengths observed/reported: **3, 4, 5, 6, and 7 digits**.

## 8.3 Candidate highlighting

**Observed**

Selecting a number highlights **all same-size spaces** orange.

This highlighting is a geometric/length affordance. It does not mean the selected number is correct for every highlighted slot.

## 8.4 Digit order

**Observed**

- Horizontal: first digit at left, proceeding left → right.
- Vertical: first digit at top, proceeding top → bottom.
- Numbers are not reversed by the player.

## 8.5 Orientation

**Observed**

Orientation is automatic. The game rotates/reorients the selected piece to the slot's available orientation. There is no manual piece-rotation control.

## 8.6 Intersections

**Observed**

At a valid intersection:

```text
horizontal digit == vertical digit == locked game digit (if one exists)
```

When player numbers disagree, the new placement wins and the contradictory older player number is evicted.

## 8.7 Slot occupancy

**Observed**

- At most one player number occupies a given slot at a time.
- An occupied slot may be targeted by another same-length piece.
- Committing the incoming piece replaces and removes the old slot occupant.

## 8.8 Cell ownership is not permanent for player pieces

**Observed**

A white player-filled cell is only present because its current player number is placed. Removing that piece removes its contribution from the board unless another remaining piece or a locked game digit also supplies the cell.

---

# 9. Placement resolution algorithm — behavior contract

This is a gameplay contract, not implementation code.

When the player releases a piece onto a same-length target slot:

1. **Accept the target slot.** Contradicting player digits do not make the placement illegal.
2. If the target slot already contains another player piece, **evict that target-slot occupant**.
3. Compare the incoming number with every crossing player piece at shared cells.
4. For each mismatch, **evict the contradictory crossing player piece**.
5. Preserve every crossing player piece whose shared digit matches.
6. Preserve every locked game-supplied blue cell.
7. Assign the incoming piece to the target slot.
8. Recompute visible digits/cell ownership.
9. Return every evicted piece to the available bank.
10. Save the new in-level state.
11. Re-evaluate the win condition.

### Exception

The exact resolution when the incoming piece contradicts a locked blue cell is **Unknown** and must not be invented as original behavior until clarified.

---

# 10. Feedback/color state specification

## 10.1 Neutral board

- Blocked cell: dark navy.
- Empty active cell: white.
- Game-supplied locked digit cell: light blue.
- Settled player-filled digit cell: white.
- Normal digit text: dark navy.

## 10.2 Candidate state

**Observed**

- Select a number → every same-length slot glows orange.
- Selected bank/category state also receives orange/yellow emphasis in the captured UI.

## 10.3 Valid target state

**Observed**

When the selected number fits and introduces no contradiction:

- target space glows green;
- selected number glows green.

## 10.4 Contradiction state

**Observed**

When an intersection digit disagrees:

- contradicting digit is red/pink;
- conflicting number/space receives red/pink glow;
- red is warning feedback, not a placement blocker.

## 10.5 Occupied-slot replacement state

**Observed**

When hovering a new same-length number on a slot already holding another player number:

- incoming number glows orange;
- existing slot number glows orange;
- releasing replaces the existing number.

## 10.6 Glow implementation

**Observed visually, exact animation unknown**

Screenshots establish colored highlighted/glowing states but not exact shader, pulse frequency, opacity tween, or duration. Match appearance closely; do not claim an exact original animation curve.

---

# 11. Number bank

## 11.1 Grouping

**Observed**

Numbers appear below the puzzle area grouped by digit count.

Observed group labels include:

- `3 digits`
- `4 digits`
- `5 digits`
- `6 digits`
- `7 digits`

## 11.2 Piece identity and availability

**Observed / implementation consequence**

- Unplaced player pieces are available in the bank.
- A piece currently placed on the board is not simultaneously available as another copy in the bank.
- If a placed piece is picked up/removed or evicted by another placement, it becomes available again.
- Blue game-supplied digits are cells/constraints; they are not removable player pieces.

## 11.3 Data representation

**Inferred implementation requirement**

Store every number as a **digit string**, not a numeric value. Positional digits matter; arithmetic magnitude does not. This also safely preserves zeros and any future leading zeros.

Each physical piece should have a stable identity independent of text in case duplicate-valued pieces ever exist.

### Unknown

- Whether duplicate-valued pieces occur.
- Whether any authored piece begins with `0`.

---

# 12. Validation model

The game has **feedback validation**, not fail/reject validation, for contradictions between player-placed numbers.

## 12.1 Length validation

**Observed**

Only same-length slots become candidates for a selected number. A number is not placed into a differently sized slot.

## 12.2 Intersection validation

**Observed**

The game compares digits at every crossing in real time while targeting/hovering.

- Match → green compatible feedback.
- Mismatch → red/pink contradiction feedback.

## 12.3 Contradictions are recoverable

**Observed**

A mismatching placement may still be committed. The game resolves the contradiction by removing the conflicting older player number.

There is no mistake counter or loss penalty.

## 12.4 Replacing an occupied slot

**Observed**

Placing a new piece into a slot already occupied by another player piece removes the old piece and leaves the new piece in that slot.

---

# 13. Win condition

## Observed

The win screen shows the completed puzzle with no active contradictions and no remaining placement task.

## Strongly inferred implementation rule

Trigger completion when all required player-number slots are simultaneously occupied in a fully consistent state and all locked game digits are satisfied.

Equivalent checks for the supplied authored puzzles are expected to be:

- every required slot has a player piece or is otherwise completely game-satisfied as authored;
- every required player piece is placed exactly once;
- all shared digits agree;
- all locked game-supplied digits agree with the occupying number(s);
- no unresolved empty required cells remain.

Because contradictory player pieces are automatically evicted, a completely occupied board should naturally be intersection-consistent.

## Inferred

If an authored puzzle permits more than one fully constraint-valid arrangement, any complete valid arrangement should count as solved. The mechanic is constraint satisfaction rather than arithmetic evaluation.

## Unknown

The original game's internal completion implementation (canonical slot assignment vs pure constraint check) cannot be observed directly.

---

# 14. Progression and persistence

## 14.1 Level sequence

**Observed**

- Level 1 is a tutorial.
- Later levels become larger/more complex and use several number lengths.
- Previous completed levels **cannot be played again**.
- The case-study brief requires at least 5 playable levels.

**Case-study authored extension**

This implementation uses **6 linear levels**:

1. Level 1 — supplied tutorial.
2. Level 2 — supplied/reconstructed 6×6 puzzle.
3. Level 3 — supplied/reconstructed 8×8 contradiction-example puzzle.
4. Level 4 — authored 9×9 puzzle.
5. Level 5 — authored 10×10 puzzle.
6. Level 6 — authored 11×11 puzzle.

Levels 4–6 add no new mechanics. Difficulty rises only through layout, slot count, intersection count, starting clue density, and same-length ambiguity.

## 14.2 Difficulty progression

**Observed / inferred**

Complexity increases by introducing:

- larger grids;
- more number pieces;
- more intersections;
- more digit-length categories;
- more candidate slots of equal size;
- increased need to resolve/rearrange conflicts.

**Case-study authored difficulty curve**

| Level | Grid | Slots | Intersections | Authored starting blue clues | Main difficulty change |
|---|---:|---:|---:|---:|---|
| 1 | 5×5 | 2 | 1 | 2 | Tutorial: one horizontal + one vertical placement |
| 2 | 6×6 | 6 | 6 | Source-defined | Multiple pieces and two length groups |
| 3 | 8×8 | 10 | 10 | Source-defined captured state | 3–7 digit groups and conflict/eviction pressure |
| 4 | 9×9 | 12 | 18 | 10 | More crossings; every length 3–7 appears |
| 5 | 10×10 | 15 | 24 | 9 | More same-length candidates and lower clue density |
| 6 | 11×11 | 19 | 32 | 8 | Highest crossing density; eight 7-digit pieces create heavy candidate ambiguity |

The authored extension intentionally reduces the proportion of starting blue clues from Level 4 onward while increasing intersections and repeated piece lengths.

## 14.3 In-level save

**Observed**

Current level progress persists if the user leaves the game and returns.

The player can then choose either:

- Continue the saved board, or
- Restart the current level.

### Minimum persisted state

To reproduce the experience, persist:

- current progression level ID/index;
- each player piece's current slot or unplaced state;
- hint-created locked cells;
- any hint balance that is actually progression/session-persistent once its scope is clarified;
- relevant settings such as Sound/Vibration.

## 14.4 Completed levels

**Observed**

Once a level is completed and progression advances, the player cannot go back and replay previous levels.

Therefore do **not** build a normal level-select/replay screen unless introduced explicitly as a case-study extension.

## 14.5 Main/New Game behavior

**Strongly inferred from observed progression**

- First-ever play starts Level 1.
- With no unfinished saved attempt, New Game starts the current next uncompleted level.
- With unfinished progress, entry must permit Continue or Restart.
- After win, New Game proceeds to the next progression level.

### Unknown

- Behavior after the final available level.
- Exact UI presentation of Continue/Restart.

---

# 15. Hint system

## Observed

- Standard gameplay has a bulb button.
- Captured badge value is `5`.
- Using Hint fills one **random empty digit cell** with the correct digit.
- It does not place an entire number.
- The newly filled digit is game-supplied and therefore locked/non-removable and light blue.

## Inferred

- Badge likely represents remaining hints.
- Hint use likely decrements the displayed count.

## Unknown

- Hint balance scope: per level, per session, global account balance, etc.
- Whether hints persist/replenish across progression.
- Exact random-selection policy.
- Whether restart restores hints consumed on the restarted attempt.

---

# 16. Restart and Back

## Restart

**Observed**

Restart is always available in standard gameplay and is independent of failure because the game has no failure state.

Functional reset behavior should restore the level's authored initial puzzle state.

## Back

**Observed**

A Back control exists in standard gameplay.

Because in-level progress is saved, leaving gameplay must not automatically discard the current arrangement.

### Unknown

- Exact destination after Back.
- Whether a leave confirmation appears.

---

# 17. Level maps and authored puzzle data

Notation:

- `#` = blocked cell
- `.` = empty active cell
- digit = game-supplied locked light-blue digit in the initial/captured state

Rows/columns are 1-indexed.

Levels 1–3 are based on supplied evidence. Levels 4–6 are **case-study-authored extension levels** created from the established rules only; they are not claimed to be original-game levels.

---

## 17.1 Level 1 — tutorial, exact reconstruction

### Initial grid

```text
# # 5 # #
# # . # #
1 . . . .
# # . # #
# # . # #
```

### Bank

- 5 digits: `12345`, `54321`

### Slots

- H1: row 3, columns 1–5, length 5.
- V1: column 3, rows 1–5, length 5.
- Intersection: row 3, column 3.

### Solution

- H1 = `12345`
- V1 = `54321`
- shared center digit = `3`

### Tutorial behavior demonstrated

- Selecting a 5-digit piece identifies the 5-cell placement space.
- `12345` is shown horizontally when targeting H1.
- After placement, its player-added cells are white while the original game-supplied `1` remains light blue.
- `54321` automatically presents vertically when targeting V1.
- Both numbers agree on the shared center digit `3`.

---

## 17.2 Level 2 — exact solved layout reconstructed from screenshots

### Captured topology/state

```text
. 3 . . 4 .
# 5 # # 2 #
# 7 # . 9 .
. 0 . # 1 #
# 4 # # 5 #
. 0 . . 6 .
```

### Bank

- 3 digits: `301`, `792`
- 6 digits: `309165`, `357040`, `429156`, `734748`

### Slots

- H1: r1 c1–c6, length 6.
- H2: r3 c4–c6, length 3.
- H3: r4 c1–c3, length 3.
- H4: r6 c1–c6, length 6.
- V1: c2 r1–r6, length 6.
- V2: c5 r1–r6, length 6.

### Exact solved grid shown on win screen

```text
7 3 4 7 4 8
# 5 # # 2 #
# 7 # 7 9 2
3 0 1 # 1 #
# 4 # # 5 #
3 0 9 1 6 5
```

### Slot assignment

- H1 = `734748`
- H2 = `792`
- H3 = `301`
- H4 = `309165`
- V1 = `357040`
- V2 = `429156`

The captured level also demonstrates multiple digit-length categories and same-length candidate highlighting.

---

## 17.3 Level 3 — supplied 8×8 contradiction-example puzzle

This is the previously documented **Later Puzzle A** screenshot.

### Bank

- 3 digits: `239`, `892`
- 4 digits: `1002`, `6643`
- 5 digits: `13949`, `90409`
- 6 digits: `105522`, `867122`
- 7 digits: `2103302`, `2329390`

### Captured grid state

```text
# # . . . . 2 .
# 2 # . # # 1 #
. 3 . . . # 0 #
# 2 # . # . 3 .
. 9 . # . # 3 #
# 3 # . . . 0 .
# 9 # # . # 2 #
. 0 . . . . # #
```

### Slots

Horizontal:

- H1: r1 c3–c8, length 6.
- H2: r3 c1–c5, length 5.
- H3: r4 c6–c8, length 3.
- H4: r5 c1–c3, length 3.
- H5: r6 c4–c8, length 5.
- H6: r8 c1–c6, length 6.

Vertical:

- V1: c2 r2–r8, length 7.
- V2: c4 r1–r4, length 4.
- V3: c5 r5–r8, length 4.
- V4: c7 r1–r7, length 7.

### Captured mismatch

- `892` is being targeted at H3.
- H3's middle cell crosses V4.
- V4 currently contributes `3` there.
- `892` contributes `9` there.
- The contradiction is shown in pink/red.
- The other 3-cell slot H4 remains orange as another same-length candidate.

### Correct behavioral interpretation after player clarification

This red state is **not** a rejected-placement preview.

If the player commits `892` to H3 anyway:

- `892` remains in H3;
- the contradictory player-placed V4 number is removed/evicted;
- that removed 7-digit piece becomes available again.

### Deterministically inferred full solution

- H1 = `867122`
- H2 = `13949`
- H3 = `239`
- H4 = `892`
- H5 = `90409`
- H6 = `105522`
- V1 = `2329390`
- V2 = `6643`
- V3 = `1002`
- V4 = `2103302`

This final assignment is inferred from the bank and crossings rather than directly shown in a completed screenshot.

---

## 17.4 Level 4 — authored 9×9 extension

**Status:** Case-study-authored. No gameplay changes.

### Difficulty intent

- 12 pieces.
- 18 intersections.
- All lengths 3–7 represented.
- 10 starting locked blue clues.
- More repeated 6- and 7-digit candidates than earlier levels.

### Initial grid

```text
# . # # . # # # .
# 7 . . . . 6 # .
# . # # . # . . 9
# . . . 2 . . # .
# . # # . # . # .
# . # # . 3 . # #
. 2 . . 7 . 7 # #
# # . # # . # # #
# . 6 . . 5 . . #
```

### Bank

- 3 digits: `829`, `431`, `806`
- 4 digits: `3615`
- 5 digits: `73951`
- 6 digits: `773716`, `784268`, `688717`
- 7 digits: `6282767`, `5689574`, `7787422`, `8722847`

### Slots

Horizontal:

- H1: r2 c2–c7, length 6.
- H2: r3 c7–c9, length 3.
- H3: r4 c2–c7, length 6.
- H4: r6 c5–c7, length 3.
- H5: r7 c1–c7, length 7.
- H6: r9 c2–c8, length 7.

Vertical:

- V1: c2 r1–r7, length 7.
- V2: c3 r7–r9, length 3.
- V3: c5 r1–r7, length 7.
- V4: c6 r6–r9, length 4.
- V5: c7 r2–r7, length 6.
- V6: c9 r1–r5, length 5.

### Canonical slot assignment

- H1 = `773716`
- H2 = `829`
- H3 = `784268`
- H4 = `431`
- H5 = `6282767`
- H6 = `5689574`
- V1 = `7787422`
- V2 = `806`
- V3 = `8722847`
- V4 = `3615`
- V5 = `688717`
- V6 = `73951`

### Solution grid

```text
# 7 # # 8 # # # 7
# 7 7 3 7 1 6 # 3
# 8 # # 2 # 8 2 9
# 7 8 4 2 6 8 # 5
# 4 # # 8 # 7 # 1
# 2 # # 4 3 1 # #
6 2 8 2 7 6 7 # #
# # 0 # # 1 # # #
# 5 6 8 9 5 7 4 #
```

---

## 17.5 Level 5 — authored 10×10 extension

**Status:** Case-study-authored. No gameplay changes.

### Difficulty intent

- 15 pieces.
- 24 intersections.
- All lengths 3–7 represented.
- 9 starting locked blue clues.
- Six 5-digit candidates and five 7-digit candidates increase slot ambiguity.

### Initial grid

```text
# . # # # . # . # #
# . # . . . . 4 . .
# . # # # . # . # #
2 . . . # 8 . . . .
. . 8 # # . # . # #
. 7 . . . . 3 # . #
. . . 6 # # . # . #
. # 2 . . . . . 2 #
. # # . # # . # . #
. # # . # # . # . #
```

### Bank

- 3 digits: `598`
- 4 digits: `2031`, `8926`
- 5 digits: `83593`, `38922`, `36951`, `32826`, `84456`, `32263`
- 6 digits: `903876`
- 7 digits: `8907427`, `1793163`, `2932842`, `2518043`, `3330979`

### Slots

Horizontal:

- H1: r2 c4–c10, length 7.
- H2: r4 c1–c4, length 4.
- H3: r4 c6–c10, length 5.
- H4: r5 c1–c3, length 3.
- H5: r6 c1–c7, length 7.
- H6: r7 c1–c4, length 4.
- H7: r8 c3–c9, length 7.

Vertical:

- V1: c1 r4–r10, length 7.
- V2: c2 r1–r7, length 7.
- V3: c3 r4–r8, length 5.
- V4: c4 r6–r10, length 5.
- V5: c6 r1–r6, length 6.
- V6: c7 r6–r10, length 5.
- V7: c8 r1–r5, length 5.
- V8: c9 r6–r10, length 5.

### Canonical slot assignment

- H1 = `8907427`
- H2 = `2031`
- H3 = `83593`
- H4 = `598`
- H5 = `1793163`
- H6 = `8926`
- H7 = `2932842`
- V1 = `2518043`
- V2 = `3330979`
- V3 = `38922`
- V4 = `36951`
- V5 = `903876`
- V6 = `32826`
- V7 = `84456`
- V8 = `32263`

### Solution grid

```text
# 3 # # # 9 # 8 # #
# 3 # 8 9 0 7 4 2 7
# 3 # # # 3 # 4 # #
2 0 3 1 # 8 3 5 9 3
5 9 8 # # 7 # 6 # #
1 7 9 3 1 6 3 # 3 #
8 9 2 6 # # 2 # 2 #
0 # 2 9 3 2 8 4 2 #
4 # # 5 # # 2 # 6 #
3 # # 1 # # 6 # 3 #
```

---

## 17.6 Level 6 — authored 11×11 extension

**Status:** Case-study-authored. No gameplay changes.

### Difficulty intent

- 19 pieces.
- 32 intersections.
- All lengths 3–7 represented.
- 8 starting locked blue clues.
- Eight 7-digit candidates produce the strongest same-length ambiguity in the project.
- Highest intersection count and lowest authored clue density of the extension levels.

### Initial grid

```text
# . . 4 . . . . # . #
# # # . # . # . # . #
. . . . # . # . . 5 .
# # # . # . # . # # #
# # # . 6 . . . . # #
. . 4 . . . # . . . 4
# # . . . . . 2 . # .
. # . # . # # # . # .
. # . # . . . . 4 . .
. # . # . # # # . # .
. . 7 . . # # . . . .
```

### Bank

- 3 digits: `385`
- 4 digits: `3941`, `3950`, `6554`, `1560`, `3529`
- 5 digits: `92706`
- 6 digits: `267616`, `584098`, `490607`, `429730`
- 7 digits: `7744164`, `9369524`, `1183467`, `4415203`, `6961176`, `1683789`, `4536162`, `6541475`

### Slots

Horizontal:

- H1: r1 c2–c8, length 7.
- H2: r3 c1–c4, length 4.
- H3: r3 c8–c11, length 4.
- H4: r5 c4–c9, length 6.
- H5: r6 c1–c6, length 6.
- H6: r6 c8–c11, length 4.
- H7: r7 c3–c9, length 7.
- H8: r9 c5–c11, length 7.
- H9: r11 c1–c5, length 5.
- H10: r11 c8–c11, length 4.

Vertical:

- V1: c1 r8–r11, length 4.
- V2: c3 r6–r11, length 6.
- V3: c4 r1–r7, length 7.
- V4: c5 r5–r11, length 7.
- V5: c6 r1–r7, length 7.
- V6: c8 r1–r7, length 7.
- V7: c9 r5–r11, length 7.
- V8: c10 r1–r3, length 3.
- V9: c11 r6–r11, length 6.

### Canonical slot assignment

- H1 = `7744164`
- H2 = `3941`
- H3 = `3950`
- H4 = `267616`
- H5 = `584098`
- H6 = `6554`
- H7 = `9369524`
- H8 = `1183467`
- H9 = `92706`
- H10 = `1560`
- V1 = `3529`
- V2 = `490607`
- V3 = `4415203`
- V4 = `6961176`
- V5 = `1683789`
- V6 = `4536162`
- V7 = `6541475`
- V8 = `385`
- V9 = `429730`

### Solution grid

```text
# 7 7 4 4 1 6 4 # 3 #
# # # 4 # 6 # 5 # 8 #
3 9 4 1 # 8 # 3 9 5 0
# # # 5 # 3 # 6 # # #
# # # 2 6 7 6 1 6 # #
5 8 4 0 9 8 # 6 5 5 4
# # 9 3 6 9 5 2 4 # 2
3 # 0 # 1 # # # 1 # 9
5 # 6 # 1 1 8 3 4 6 7
2 # 0 # 7 # # # 7 # 3
9 2 7 0 6 # # 1 5 6 0
```

---

# 18. Visual/UI specification

## 18.1 Reference layout

- Portrait reference: 945×2048.
- Clean mobile-first layout.
- Board centered in upper/middle area.
- Number groups below board.
- Rounded controls/cards.
- Dark navy typography.

## 18.2 Approximate palette from screenshots

These are visual approximations, not original source tokens:

- Dark navy text / blocked cells: roughly `#2B4057`–`#30465F`.
- Game-supplied locked blue: roughly `#C9DCEF`.
- Neutral bank header slate-blue: roughly `#8EA2B7`.
- Primary action blue: roughly `#2687F5`.
- Candidate/replacement orange-yellow: warm pale orange/yellow.
- Valid match green: pale mint/green.
- Contradiction red: pale salmon/pink-red.
- Screen background: very pale blue-gray.
- Win background: saturated bright/royal blue.
- Congratulations: yellow/orange with darker edge/shadow.

## 18.3 Grid rendering

**Observed**

- Square cells.
- Dark internal grid lines.
- Thick dark outer border with subtly rounded outer corners.
- Large centered digits.
- Blocked cells filled dark navy.

## 18.4 Bank cards

**Observed**

- Rounded white bodies with subtle shadow.
- Colored header with centered bold white digit-count label.
- Numbers arranged with generous spacing.
- Selected category receives orange/yellow feedback.

## 18.5 Animation/motion directly evidenced

**Observed state changes**

- Selected pieces move independently of their source list.
- Piece orientation changes automatically to suit horizontal/vertical target spaces.
- Orange, green, and red/pink glow states exist.
- Confetti appears on win.

**Unknown exact motion parameters**

- drag smoothing;
- snap tween/easing;
- rotation tween duration;
- glow pulse/tween parameters;
- eviction animation;
- return-to-bank animation;
- win-screen transition timing;
- confetti speed/duration;
- button press micro-animation.

A faithful recreation should match the visible states first and use restrained transitions where exact timing is unavailable.

---

# 19. Audio and haptics

## Observed

- Sound toggle exists.
- Vibration toggle exists.
- Both are shown enabled in Settings.

## Unknown

The supplied evidence does not identify exactly which events trigger audio or haptics. Do not claim original parity for specific pickup, match, contradiction, replacement, hint, restart, or win feedback without further evidence.

---

# 20. Required implementation data model

This section defines the conceptual data needed; it is not source code.

## 20.1 Level

Each level needs:

- stable level ID/index;
- grid rows/columns;
- blocked/active topology;
- authored initial game-supplied locked digits;
- list of slots;
- list of player number pieces;
- solution digit for each active cell, used by Hint and completion validation;
- optional tutorial/tool-visibility configuration;
- progression relationship to the next level.

## 20.2 Slot

Each slot needs:

- stable slot ID;
- orientation: horizontal or vertical;
- ordered cell-coordinate list;
- length;
- current player piece ID or empty state.

## 20.3 Piece

Each player piece needs:

- stable piece ID;
- digit string;
- digit count;
- current state: bank / selected / placed;
- assigned slot ID if placed.

Eviction/removal returns the piece to the bank state.

## 20.4 Cell

Each cell needs:

- coordinate;
- blocked vs active;
- correct solution digit;
- gameLocked flag;
- game-supplied digit if locked;
- current effective display digit;
- horizontal slot membership if any;
- vertical slot membership if any;
- transient visual feedback state: neutral / candidate-orange / compatible-green / contradiction-red / replacement-orange.

## 20.5 Save state

Persist at least:

- current progression level;
- player piece-to-slot assignments;
- hint-created locked cells;
- settings;
- hint count/balance if its scope is persistent.

Do not persist transient hover/glow states.

---

# 21. Core implementation acceptance criteria

A recreation agent should not consider the base game complete until all of these work:

1. Render authored rectangular grids with active and blocked cells.
2. Render game-supplied locked digits in light blue.
3. Render player-filled digits on white cells.
4. Show number pieces below the board grouped by digit count from 3–7 digits as applicable.
5. Support selecting/dragging an unplaced number from the bank.
6. Support selecting/picking up a player-placed number from the board.
7. Automatically orient the selected number horizontally or vertically for the target slot; no manual rotation control.
8. Highlight **all same-length slots** orange when a number is selected.
9. Show selected number + compatible target slot green when no digit contradiction exists.
10. Detect intersection contradictions while targeting and show red/pink feedback.
11. Permit placement despite contradiction between player pieces.
12. On such placement, automatically remove every contradictory older player number and return it to the bank.
13. Allow targeting an already occupied same-length slot.
14. Show incoming and existing same-slot numbers orange during replacement hover.
15. On replacement, remove the old occupant and commit the incoming number.
16. Allow every player-placed number to be picked up, moved, and removed from its slot.
17. Never allow player interaction to remove a locked light-blue game digit.
18. Hint fills exactly one random empty active cell with its correct solution digit and locks it light blue.
19. Provide Restart with no failure-state requirement.
20. Save in-level progress between sessions.
21. On re-entry to an unfinished level, provide Continue vs Restart behavior.
22. Prevent replay of previously completed levels in normal progression.
23. Trigger win when the puzzle is completely and consistently filled.
24. Show Congratulations screen, completed grid, confetti, New Game, and Main.
25. Provide Main and Settings screens matching the supplied visible structure.
26. Include six playable levels in this case-study implementation.
27. Do **not** introduce arithmetic equations as a core mechanic.

---

# 22. Monthly Pass — case-study extension, not original reverse-engineered gameplay

The master instructions require a new Monthly Pass. It is **not** visible in the original screenshots and must remain conceptually separate from the reverse-engineered base game.

## Observed requirements from the master instructions

- 30-day reward track.
- Free tier.
- Premium tier.
- Progress earned by completing levels.
- Rewards are claimable.
- Pass progress persists between sessions.
- Real in-app purchases are not required.
- Premium unlock is mocked.

## Still-undesigned pass details

The supplied original game provides no economy/currency inventory to copy, so the following remain new-design decisions:

- number of pass milestones;
- progress earned per completed level;
- reward contents;
- premium reward contents;
- retroactive premium claims;
- claim interaction;
- pass entry point in the UI;
- 30-day reset/time handling;
- premium mock price/purchase presentation.

These should be designed in a separate pass and labeled as additions rather than original-game behavior.

---

# 23. Master Observed / Inferred / Unknown table

| Topic | Observed | Inferred | Unknown |
|---|---|---|---|
| Core mechanic | Same-length number placement + crossing digit matching | Constraint-satisfaction puzzle | — |
| Arithmetic | No arithmetic mechanic | Brief's “balancing” maps to digit consistency | — |
| Failure | No fail condition | Player can iterate indefinitely | — |
| Blue cells | Put there by game; locked; non-removable | Hint-created digit also uses same ownership/style | Locked-digit mismatch resolution |
| White cells | Player-put digits | Shared player intersections remain white unless game-locked | — |
| Piece lengths | 3–7 digits | — | Leading-zero pieces / duplicates |
| Candidate slots | All same-size spaces orange | Length-based regardless of correctness | Exact orange tween |
| Orientation | Automatic H/V | No manual rotate needed | Exact free-drag orientation policy if several orientations available |
| Valid hover | Space + number green | Indicates no contradiction | Exact animation timing |
| Contradiction | Digit/space/number red or pink | Warning, not rejection | Color precedence during simultaneous replacement + contradiction |
| Contradicting placement | Allowed | Incoming piece wins | Locked-blue mismatch behavior |
| Conflict resolution | Contradictory older player number removed | Multiple contradictory older numbers all evicted | Exact eviction animation |
| Occupied slot | Incoming + existing numbers orange | Replacement state | Color precedence with cross-conflicts |
| Occupied commit | New number replaces old | Old returns to bank | Exact movement animation |
| Placed pieces | Can be picked up and removed | Can be relocated between same-length slots | Exact removal gesture |
| Hint | Fills one random empty digit | Uses correct solution digit; badge likely decrements | Hint count scope/refill/restart semantics |
| Restart | Available at will | Restores authored initial level | Hint refund + confirmation UI |
| Save | In-level progress saved | Persist piece assignments + hint cells | Exact save trigger/format |
| Return flow | Continue or Restart current level | Prompt/modal/intermediate screen likely | Exact UI presentation |
| Progression | L1 tutorial; completed previous levels cannot replay | Linear next-uncompleted progression | Final-level behavior |
| Win | Congratulations, confetti, completed grid | All slots/cells consistently completed | Canonical vs alternate valid solution internal check |
| Settings | Sound, Vibration, Help, Privacy, Done | Done returns | Exact audio/haptic event mapping |
| Monthly Pass | Required new feature | Separate extension | Reward/economy design |

---

# 24. Remaining implementation-material questions

The following cannot be determined from the screenshots, master instructions, or clarified gameplay description and materially affect accurate reproduction:

1. **What happens if a selected number contradicts a locked light-blue game digit?** Because blue digits cannot be removed, does the game refuse that placement, prevent targeting that slot, keep the number unplaced, or use another rule?
2. **What is the exact gesture for removing a player-placed number without immediately moving it to another slot?** For example: drag it back to the bank, release it off-grid, tap to return, or another interaction?
3. **How is the Hint count scoped?** Is the `5` shown in the screenshot five hints per level, a persistent shared balance, or something else? Does Restart refund hints used during that attempt?
4. **What exactly is the Continue/Restart UI when returning to an unfinished level?** The behavior is known, but the screen/modal/button presentation is not supplied.
5. **What happens after Level 6 is completed?** Level 6 is the final authored level in this case-study set; the end-of-content behavior and meaning of New Game remain unspecified.

Everything else required for the core number-placement experience is sufficiently specified to begin implementation without inventing arithmetic or a failure mechanic.

---

# 25. Case-study constraints from `original manual.txt`

The implementation must additionally satisfy these deliverable-level requirements:

- Unity 2022 LTS or newer.
- Core grid-based number placement.
- Intersection validation.
- “Equation balancing” implemented here as crossing-digit consistency, per clarified original behavior.
- At least 5 playable levels are required by the brief; this specification defines 6.
- Playable WebGL build.
- Documentation of the AI-assisted development process.
- Short rationale for Monthly Pass design decisions.
- Monthly Pass: 30 days, free/premium tiers, progress from level completion, claimable rewards, persisted pass progress, mocked premium unlock.

---

# 26. Do-not-invent guardrails for the coding agent

Do **not** silently add any of these as original-game behavior:

- arithmetic operators or arithmetic equations;
- failure/loss states;
- lives or mistake limits;
- score, timer, stars, streaks, leaderboard, or level ratings;
- replayable previous levels;
- a general level-select screen;
- manual piece rotation;
- rejection of player-vs-player contradictory placements;
- permanent locking of player-placed numbers;
- whole-number hints;
- currencies or inventory merely to support the Monthly Pass;
- ads or real IAP;
- procedural level generation;
- a specific unobserved animation/sound/haptic sequence.

Where Section 24 still marks behavior Unknown, implement only after deliberate clarification or document it explicitly as a case-study design choice rather than reverse-engineered fact.
