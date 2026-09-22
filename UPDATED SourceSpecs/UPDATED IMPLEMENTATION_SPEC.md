# Crossnumbers — Unity 2022 LTS+ Implementation Specification

**Source of truth:** `GAME_SPEC.md` only  
**Target:** Unity 2022 LTS or newer, WebGL-capable  
**Project goal:** Small, readable case-study implementation that can be built rapidly with AI assistance.  
**Architecture goal:** Prefer plain Unity components, serialized data, and a few focused services. Avoid frameworks, DI containers, event buses, ECS, addressables, custom render pipelines, networking, databases, or production-scale abstraction.

---

> **Visual source of truth:** use `VISUAL_DESIGN_SPEC.md` for palette, typography, line weights, spacing, responsive board sizing, and state styling.

# 1. Implementation principles

1. Keep the game data-driven enough to author all six defined levels without changing code.
2. Keep static authored data in ScriptableObjects and mutable runtime/save data in plain serializable C# classes.
3. Use only two scenes: `Main` and `Gameplay`.
4. Represent secondary screens as Canvas panels/overlays rather than separate scenes.
5. Use one persistent `GameApp` object for save/progression/pass state.
6. Keep the placement rules in one pure-ish resolver class so drag/UI code does not own gameplay logic.
7. Rebuild visible board cells from authoritative runtime assignments rather than trying to mutate cell ownership incrementally in many places.
8. Store number values as strings, never numeric types.
9. Save immediately after gameplay-affecting mutations because WebGL quit callbacks are not reliable.
10. Do not add mechanics not present in `GAME_SPEC.md`.

The five gameplay Unknowns from `GAME_SPEC.md` remain unresolved and must be isolated rather than silently invented:

- contradiction with a locked light-blue digit;
- exact removal gesture for a player-placed piece;
- hint-count scope/refund behavior;
- exact Continue/Restart presentation;
- behavior after the last authored level.

Where this document proposes a UI shell around an Unknown, it must be treated as a case-study implementation choice, not original-game behavior.

---

# 2. Unity project setup

## 2.1 Recommended project type

Use a standard Unity 2022 LTS 2D project.

Recommended built-in/package dependencies only:

- Unity UI (`UnityEngine.UI`)
- TextMeshPro
- EventSystem / legacy UI pointer interfaces
- SceneManager
- PlayerPrefs + `JsonUtility`
- ParticleSystem for win confetti
- AudioSource only if sound assets are available

Do not require:

- Input System package
- DOTween
- Zenject/VContainer
- UniRx
- Addressables
- third-party save systems
- third-party UI particle packages
- backend services

## 2.2 Canvas setup

Use a portrait-first `CanvasScaler`:

- UI Scale Mode: `Scale With Screen Size`
- Reference Resolution: `945 x 2048`
- Screen Match Mode: `Match Width Or Height`
- Match: approximately `0.5`

Put all interactive UI inside a `SafeAreaRoot` RectTransform controlled by a tiny `SafeAreaFitter` component so mobile/browser insets do not obscure controls.

---

# 3. Scene architecture

Use exactly two scenes.

## 3.1 `Main` scene

Contains:

- original Main screen;
- Settings panel;
- Continue/Restart panel;
- Monthly Pass panel;
- optional Help/Privacy shells if content/URL is supplied later.

This scene is the application entry scene.

## 3.2 `Gameplay` scene

Contains:

- gameplay toolbar;
- board;
- number bank;
- drag layer;
- win overlay;
- win confetti.

The win screen is an overlay in the Gameplay scene so the completed board can remain visible without reconstructing it in another scene.

## 3.3 Persistent application object

Create a prefab named `GameApp` and place it in the Main scene.

`GameApp` calls `DontDestroyOnLoad` and owns references to:

- `GameConfig`
- `SaveService`
- `ProgressionService`
- `MonthlyPassService`

Use a simple singleton guard so only one `GameApp` exists after scene reloads.

Do not build a generic service locator. Components that need these services may access `GameApp.Instance` directly; this is acceptable for this case-study scale.

---

# 4. Suggested Assets/ folder structure

```text
Assets/
  Art/
    UI/
    Icons/
    Confetti/
  Audio/
  Fonts/
  Prefabs/
    Core/
      GameApp.prefab
    Gameplay/
      GridCell.prefab
      NumberPiece.prefab
      BankGroup.prefab
    MonthlyPass/
      PassMilestone.prefab
  Scenes/
    Main.unity
    Gameplay.unity
  ScriptableObjects/
    GameConfig.asset
    Levels/
      Level_01.asset
      Level_02.asset
      Level_03.asset
      Level_04.asset
      Level_05.asset
      Level_06.asset
    MonthlyPass/
      MonthlyPassConfig.asset
  Scripts/
    Core/
    Data/
    Main/
    Gameplay/
    MonthlyPass/
    UI/
```

`Level_01`–`Level_03` are source-derived/reconstructed. `Level_04`–`Level_06` are explicit case-study-authored extension levels using the same rules and no new mechanics. Do not claim Levels 4–6 came from the original game.

---

# 5. ScriptableObjects

Use three static configuration assets.

## 5.1 `GameConfig : ScriptableObject`

Purpose: ordered catalog and global references.

Fields:

```text
List<LevelDefinition> levels
MonthlyPassConfig monthlyPass
```

The list order is the linear progression order. Do not build a separate level-select catalog.

## 5.2 `LevelDefinition : ScriptableObject`

One asset per level.

Fields:

```text
string levelId
int rows
int columns
bool isTutorial
bool showToolbar

List<CellCoord> blockedCells
List<LockedCellDefinition> initialLockedCells
List<SlotDefinition> slots
List<PieceDefinition> pieces
List<SolutionCellDefinition> solutionCells
```

### `CellCoord`

```text
int row
int column
```

Use zero-based coordinates internally.

### `LockedCellDefinition`

```text
CellCoord coord
char digit
```

### `SlotDefinition`

```text
string slotId
SlotOrientation orientation   // Horizontal | Vertical
CellCoord start
int length
```

Derive the ordered cell coordinates at runtime from `start`, `orientation`, and `length`. This is simpler to author than serializing every cell coordinate.

### `PieceDefinition`

```text
string pieceId
string digits
```

`digits.Length` is the piece length. Keep a stable ID even when digit strings are unique.

### `SolutionCellDefinition`

```text
CellCoord coord
char digit
```

This map is used by Hint. It should contain every active solution cell.

Do not use numeric types for piece values because positional digits, zeroes, and potential future leading zeroes matter.

## 5.3 `MonthlyPassConfig : ScriptableObject`

This is a case-study extension, not reverse-engineered original-game content.

Fields:

```text
string seasonId
long startUtcTicks
int durationDays = 30
int progressPerCompletedLevel
List<PassMilestoneDefinition> milestones
```

### `PassMilestoneDefinition`

```text
string milestoneId
int requiredProgress
PassRewardDisplay freeReward
PassRewardDisplay premiumReward
```

### `PassRewardDisplay`

```text
string rewardId
string displayName
Sprite icon
```

The source spec does not define an economy or reward utility. Therefore the minimal implementation should make claiming a reward persist its claimed state and update the UI, but should not invent currencies, inventory, boosters, or hint grants. If functional reward effects are later required, extend `PassRewardDisplay` deliberately.

The source spec also does not define milestone count, milestone thresholds, or reward contents. These remain authored configuration, not hardcoded behavior.

---

# 6. Plain C# runtime data models

Do not mutate ScriptableObject assets during play.

## 6.1 `LevelRuntimeState`

```text
string levelId
Dictionary<string, string> slotToPieceId
Dictionary<string, string> pieceToSlotId
HashSet<CellCoord> hintedLockedCells
```

Do not serialize the dictionaries directly. They are runtime convenience structures built from save DTO lists.

The runtime level state should be the authoritative gameplay model.

## 6.2 `DragContext`

Transient only; never save.

```text
string pieceId
string originSlotId   // null if from bank
string hoveredSlotId  // null if none
List<string> previewEvictions
PlacementPreviewType previewType
```

## 6.3 `DigitClassFocusState`

Transient UI-only state; do not save it.

```text
bool active
int digitLength
string focusedSlotId             // optional; useful for crossing-cell orientation state
CellCoord lastPressedCrossingCell
bool hasLastCrossingCell
SlotOrientation crossingOrientation  // Vertical on first press, toggles on repeated same-cell presses
```

Keep this on `GameplayController` or a tiny plain C# object. It does not belong in `LevelRuntimeState` because it has no gameplay/persistence effect.

---

## 6.4 `PlacementPreviewType`

```text
None
Candidate
Compatible
Contradiction
OccupiedReplacement
LockedConflictUnknown
```

The last value exists specifically to avoid inventing the unresolved locked-blue mismatch behavior.

---

# 7. Save format

Use one JSON document stored in one PlayerPrefs key, for example:

```text
crossnumbers.save.v1
```

Use `JsonUtility`, so save DTOs should use lists rather than dictionaries or hash sets.

## 7.1 Root DTO

```text
SaveDataV1
  int schemaVersion = 1
  int currentLevelIndex
  LevelProgressSave currentLevelProgress
  SettingsSave settings
  MonthlyPassSave monthlyPass
```

## 7.2 `LevelProgressSave`

```text
string levelId
List<PiecePlacementSave> placements
List<HintedCellSave> hintedCells
bool hasStarted
```

### `PiecePlacementSave`

```text
string pieceId
string slotId
```

Only currently placed pieces need entries. Unplaced pieces are implied by absence.

### `HintedCellSave`

```text
int row
int column
char digit
```

The digit is redundant with the authored solution map but useful for defensive validation/migration.

## 7.3 `SettingsSave`

```text
bool soundEnabled = true
bool vibrationEnabled = true
```

## 7.4 `MonthlyPassSave`

```text
string seasonId
int progress
bool premiumUnlocked
List<string> claimedFreeMilestoneIds
List<string> claimedPremiumMilestoneIds
List<string> rewardedLevelIds
```

`rewardedLevelIds` prevents accidentally granting pass progress twice if a level-completion transition is replayed or a save occurs around the win flow.

## 7.5 Save rules

Call `SaveService.SaveNow()` after:

- a piece placement/replacement/eviction commit;
- a player piece removal once the final removal gesture is defined;
- a Hint;
- Restart;
- settings toggle;
- level completion/progression advance;
- Monthly Pass premium mock unlock;
- reward claim.

Also save on `OnApplicationPause(true)` and `OnApplicationFocus(false)` where applicable, but do not rely on application quit for correctness.

Restart overwrites `currentLevelProgress` with a fresh state for the same authored level.

When progression advances, previous level state may be discarded because previous levels cannot be replayed.

---

# 8. Core C# class list

Keep classes focused but small.

## 8.1 Persistent/core

### `GameApp : MonoBehaviour`

Responsibilities:

- singleton lifetime;
- references `GameConfig`;
- initializes `SaveService`, `ProgressionService`, `MonthlyPassService`;
- exposes them to both scenes.

### `SaveService`

Responsibilities:

- load/save `SaveDataV1`;
- create defaults if no save exists;
- validate `schemaVersion`;
- call `PlayerPrefs.Save()` immediately after writing.

No encryption, cloud save, database, or save slots.

### `ProgressionService`

Responsibilities:

- expose current `LevelDefinition` from `currentLevelIndex`;
- create/reset current `LevelProgressSave`;
- report whether unfinished progress exists;
- mark current level complete;
- advance to next index if one exists;
- prevent access to previous levels by never exposing a level-select operation.

Keep final-level behavior as one isolated unresolved branch because `GAME_SPEC.md` does not define it.

### `SceneNavigator`

This can be a small static helper rather than a service.

Functions:

```text
LoadMain()
LoadGameplay()
```

No scene stack system is necessary.

---

# 9. Main scene hierarchy

Suggested hierarchy:

```text
Main
  Main Camera
  EventSystem
  GameApp                    // persistent prefab
  Canvas
    SafeAreaRoot
      BackgroundDecor
      MainPanel
        Logo
        SettingsButton
        PassButton           // case-study extension
        NewGameButton
      SettingsPanel          // initially hidden
        TopBar
          Title
          DoneButton
        SoundRow
          SoundToggle
        VibrationRow
          VibrationToggle
        HelpRow
        PrivacyRow
      ResumeChoicePanel      // initially hidden
        ContinueButton
        RestartButton
      MonthlyPassPanel       // initially hidden
        Header
          Title
          CloseButton
          RemainingTimeLabel
          PremiumStatus
        ProgressBar
        MilestoneScrollView
        MockPremiumUnlockButton
      HelpPanel              // optional shell only
      PrivacyPanel           // optional shell only
```

## 9.1 `MainScreenController`

Responsibilities:

- show MainPanel by default;
- gear opens SettingsPanel;
- Pass opens MonthlyPassPanel;
- New Game checks `ProgressionService`:
  - no unfinished progress -> load Gameplay;
  - unfinished progress -> show ResumeChoicePanel.

The exact original Continue/Restart presentation is Unknown. Using a modal panel here is an explicit case-study UI choice and should be documented as such.

## 9.2 `SettingsPanelController`

Responsibilities:

- bind toggles to `SettingsSave`;
- persist changes immediately;
- Done closes panel;
- Help/Privacy call supplied destinations only when content/URL exists.

Do not invent Help copy or a Privacy URL.

---

# 10. Gameplay scene hierarchy

Suggested hierarchy:

```text
Gameplay
  Main Camera
  EventSystem
  Canvas
    SafeAreaRoot
      GameplayRoot
        Toolbar
          BackButton
          RestartButton
          Spacer
          HintButton
            HintCountBadge
        BoardRoot
          BoardBackground
          CellContainer
        BankScrollView
          Content
        DragLayer
      WinOverlay             // initially hidden
        WinBackground
        CongratulationsLabel
        CompletedBoardFrame
        NewGameButton
        MainButton
  ConfettiParticleSystem     // disabled until win
```

For Level 1, `GameplayController` hides the standard toolbar when `LevelDefinition.showToolbar == false`.

---

# 11. Gameplay prefabs

## 11.1 `GridCell.prefab`

Components:

- `RectTransform`
- `Image` background
- `TMP_Text` digit
- optional child Image for glow/overlay if easier than changing base color
- `GridCellView`

`GridCellView` only renders a supplied visual state. It should not decide gameplay rules.

Required visual states:

```text
Blocked
Empty
LockedBlue
PlayerWhite
DigitClassOrange
CandidateOrange
CompatibleGreen
ContradictionRed
ReplacementOrange
```

Use the tokens in `VISUAL_DESIGN_SPEC.md`. Store them in one serialized `GameplayVisualConfig` (or equivalent ScriptableObject) rather than hardcoding colors throughout scripts. `DigitClassOrange` and `CandidateOrange` may use the same orange token.

## 11.2 `NumberPiece.prefab`

Components:

- `RectTransform`
- background Image
- optional LayoutGroup for digit rendering
- `TMP_Text` or digit-cell children
- `CanvasGroup`
- `PieceView`

For speed, a single TMP label is acceptable in the bank. During drag/board preview, render per-digit cells if needed to match the grid.

`PieceView` implements:

- `IPointerDownHandler`
- `IBeginDragHandler`
- `IDragHandler`
- `IEndDragHandler`

No manual rotation input exists.

## 11.3 `BankGroup.prefab`

Contains:

```text
Background/Button  // whole class card is pressable
HeaderLabel        // "3 digits", etc.
PiecesContainer
```

`BankGroupView` exposes `OnDigitClassPressed(int digitLength)`. Pressing the card/header selects the digit class without selecting a number piece.

Groups are generated from the current level's unplaced pieces ordered by digit length.

## 11.4 `PassMilestone.prefab`

Contains:

- milestone threshold/progress label;
- free reward display + claim button;
- premium reward display + claim button;
- premium lock visual;
- claimed state.

No special reward inventory UI is needed.

---

# 12. Gameplay controller architecture

Use one scene-level coordinator plus small view/helper classes.

## 12.1 `GameplayController : MonoBehaviour`

Owns:

- current `LevelDefinition`;
- current `LevelRuntimeState`;
- `BoardView`;
- `BankView`;
- `PlacementResolver`;
- `HintService`;
- `WinEvaluator`;
- current `DragContext`.

Responsibilities:

1. load current level and saved progress;
2. build slot/cell lookup tables;
3. render board and bank;
4. respond to digit-class/card presses and board-space presses;
5. maintain transient crossing-cell vertical/horizontal focus cycling;
6. respond to piece drag lifecycle;
7. request placement preview from resolver;
8. commit placement result;
9. save;
10. test win;
11. show win overlay.

Avoid an event bus. Direct method calls are sufficient.

## 12.2 `BoardView`

Responsibilities:

- instantiate the grid once per level;
- map coordinates to `GridCellView`;
- draw blocked/empty/locked/player digits from a computed board snapshot;
- draw transient slot highlights;
- convert pointer position to the slot currently being targeted;
- convert a tap position to `CellCoord`;
- return all slots containing a tapped cell;
- render digit-class focus by highlighting every slot of a supplied length.

BoardView should not mutate gameplay state.

## 12.3 `BankView`

Responsibilities:

- group unplaced pieces by `digits.Length`;
- instantiate/update PieceViews;
- hide/remove a piece when placed;
- re-show evicted/removed pieces;
- show selected-category orange feedback where required;
- make each digit-class card/header pressable and notify `GameplayController`;
- clear/switch class highlight when another class, board space, or piece becomes active.

For six small levels, rebuilding the bank after each committed placement is acceptable and simpler than complex incremental bookkeeping.

---

# 13. Digit-class and board-space focus

This interaction exists independently of piece placement.

## 13.1 Pressing a digit-class card

```text
OnDigitClassPressed(length):
    CancelTransientPiecePreviewIfAny()
    focus.active = true
    focus.digitLength = length
    focus.focusedSlotId = null
    focus.hasLastCrossingCell = false

    BoardView.HighlightSlotsByLength(length, DigitClassOrange)
    BankView.HighlightDigitClass(length)
```

Do not run placement validation and do not select a piece.

## 13.2 Pressing a board cell

Create a lookup at level load:

```text
CellCoord -> List<SlotDefinition>
```

For a tapped cell:

```text
slots = slotsByCell[cell]

if slots.Count == 0:
    no class change required

if slots.Count == 1:
    FocusSlot(slots[0])
    clear crossing-cycle memory

if slots.Count == 2:
    resolve using crossing-cycle rule below
```

`FocusSlot(slot)` does:

```text
focus.active = true
focus.digitLength = slot.length
focus.focusedSlotId = slot.slotId
BoardView.HighlightSlotsByLength(slot.length, DigitClassOrange)
BankView.HighlightDigitClass(slot.length)
```

## 13.3 Crossing-cell vertical-first cycle

A normal Crossnumbers intersection belongs to one vertical and one horizontal slot.

```text
if tapped cell != focus.lastPressedCrossingCell:
    chosen = verticalSlot
    focus.crossingOrientation = Vertical
else if focus.crossingOrientation == Vertical:
    chosen = horizontalSlot
    focus.crossingOrientation = Horizontal
else:
    chosen = verticalSlot
    focus.crossingOrientation = Vertical

focus.lastPressedCrossingCell = tapped cell
focus.hasLastCrossingCell = true
FocusSlot(chosen)
```

This must use **consecutive presses on the same intersection cell**. Pressing another cell resets the next intersection press to vertical-first behavior.

If both crossing slots have equal length, still toggle `focusedSlotId`/orientation internally even though the set of highlighted same-length slots and bank class is visually identical.

## 13.4 Tap-versus-drag disambiguation

To preserve both slot inspection and moving player pieces:

- pointer down alone does not immediately remove a placed piece;
- if movement passes the normal drag threshold on a player-placed number, begin piece drag/pickup;
- if pointer up occurs without a drag, treat it as a board-cell press and run digit-class focus.

This is the simplest implementation choice consistent with both required interactions.

## 13.5 Visual precedence

When no piece is actively selected/dragged, digit-class focus uses the standard orange class/slot highlight.

When a piece becomes selected, piece placement preview supersedes digit-class focus:

```text
Contradiction / Compatible / Replacement preview
> Piece candidate orange
> Digit-class focus orange
> Settled cell state
```

Digit-class focus is transient and is not serialized.

---

# 14. Slot targeting and automatic orientation

## 14.1 Candidate discovery

When a piece becomes selected:

```text
candidateSlots = all level.slots where slot.length == piece.digits.Length
```

Highlight every candidate slot orange.

Do not pre-filter candidates by correctness.

## 14.2 Hover target selection

For each candidate slot, compute its board-space rectangle from its member cells.

The active target is the candidate slot whose rectangle contains the pointer and is nearest the pointer center if rectangles overlap at an intersection.

Because horizontal and vertical candidate rectangles can intersect, use a deterministic tie-breaker:

1. prefer the slot whose long axis best matches recent drag motion;
2. if still tied, keep the previously targeted slot;
3. if there was no prior target, choose the nearest slot center.

This is an engineering choice for stable drag behavior, not a claimed original rule.

## 14.3 Automatic orientation

Once a target slot is selected:

- Horizontal slot -> render dragged number left-to-right.
- Vertical slot -> render dragged number top-to-bottom.

No rotate button or rotate gesture.

---

# 15. Board snapshot computation

Do not store a mutable digit directly on each visual cell as the source of truth.

Whenever committed state changes, compute a `BoardSnapshot` from:

1. blocked topology;
2. initial locked cells;
3. hint-created locked cells;
4. every currently placed player piece.

Suggested snapshot cell fields:

```text
bool blocked
bool locked
char? lockedDigit
char? playerDigit
string horizontalPieceId
string verticalPieceId
```

The effective displayed digit is:

1. locked digit if locked;
2. otherwise player digit if supplied by any placed piece;
3. otherwise empty.

After normal committed placement resolution, two player pieces sharing a cell must always contribute the same digit.

This full recomputation is cheap for grids up to the observed 8x8 size and dramatically reduces bug risk.

---

# 16. Placement preview algorithm

`PlacementResolver` should be a plain C# class with no Unity UI dependencies.

Input:

```text
LevelDefinition level
LevelRuntimeState state
string incomingPieceId
string targetSlotId
```

Output:

```text
PlacementPreview
  PlacementPreviewType type
  List<string> piecesToEvict
  List<CellCoord> mismatchingCells
  bool hasLockedConflict
```

Algorithm:

1. Verify incoming piece length equals target slot length.
2. Identify the current player occupant of the target slot, if any.
3. Build the incoming piece digit at each target-slot cell.
4. Compare incoming digits against every locked game-supplied digit at those cells.
5. If any locked digit mismatches, return `LockedConflictUnknown`.
   - Do not silently define commit behavior here because `GAME_SPEC.md` explicitly marks it Unknown.
6. Find every placed crossing piece touching the target slot.
7. Compare each shared digit.
8. Add every crossing piece with at least one mismatch to `piecesToEvict`.
9. Determine preview state:
   - target slot occupied by another player piece -> `OccupiedReplacement` unless a future clarified color-precedence rule overrides it;
   - otherwise any crossing mismatch -> `Contradiction`;
   - otherwise -> `Compatible`.

All other same-length candidate slots stay orange.

The source spec leaves color precedence unknown when occupied-slot replacement and crossing contradiction happen simultaneously. Keep the precedence decision in one renderer method so it can be changed without touching placement logic.

---

# 17. Placement commit algorithm

For a target with no unresolved locked conflict:

1. If the incoming piece is currently placed in another slot, detach it from that origin slot.
2. If target slot has another piece, evict that target occupant.
3. Evict every contradictory crossing piece returned by preview.
4. Assign incoming piece to target slot.
5. Rebuild lookup dictionaries.
6. Recompute board snapshot.
7. Rebuild/update bank.
8. Clear transient candidate/hover visual states.
9. Save immediately.
10. Run win evaluation.

Eviction means only:

```text
remove piece -> slot assignment
piece becomes unplaced -> visible in bank
```

Locked game digits are never evicted.

For multiple contradictory crossing pieces, evict all of them.

---

# 18. Picking up and moving placed pieces

When a player begins dragging a board-placed piece:

1. Create `DragContext` with its origin slot.
2. Temporarily detach it from `LevelRuntimeState` for preview purposes.
3. Recompute board snapshot so its contribution disappears while the piece is being moved.
4. Show same-length candidate slots.

If committed to another slot, use the normal placement commit algorithm.

If the drag is canceled, restore the origin assignment.

The exact gesture for intentionally removing a piece to the bank is Unknown in `GAME_SPEC.md`. Keep that decision localized inside `PieceDragController.OnEndDrag` or `GameplayController.EndPieceDrag` rather than spreading assumptions across the model.

Recommended code seam:

```text
PieceDropOutcome ResolveNonSlotDrop(DragContext context)
```

Until clarified, this method should be the only place requiring a case-study UX choice.

---

# 19. Hint architecture

## 19.1 `HintService`

Input:

- current level definition;
- current board snapshot/runtime state.

Operation:

1. Build a list of active cells with no current effective digit.
2. Pick one cell randomly.
3. Read its correct digit from `LevelDefinition.solutionCells`.
4. Add the coordinate to `hintedLockedCells`.
5. Recompute board snapshot.
6. Save immediately.
7. Re-evaluate win.

Hint places one digit only.

The hinted cell becomes locked light blue and non-removable.

## 19.2 Hint count

`GAME_SPEC.md` confirms a visible `5` badge but leaves its scope/refund rules Unknown.

Do not bake a global/per-level economy into `HintService` until that decision is supplied.

Keep the visible count behind a tiny interface or method on `GameplayController`, for example:

```text
int GetDisplayedHintCount()
bool CanUseHint()
void ConsumeHint()
```

For a case-study build, the implementation team may choose a temporary configured count, but it must be documented as a new design choice, not reverse-engineered behavior.

When no empty active cell remains, Hint should do nothing and leave state unchanged; this is the safest no-op implementation, but it should not be described as confirmed original behavior.

---

# 20. Restart

`GameplayController.RestartLevel()` should:

1. discard current player placements;
2. clear hint-created locked cells;
3. preserve only authored initial locked cells;
4. reset selected/drag/highlight state;
5. rebuild board and bank;
6. save the fresh level state immediately.

Do not tie Restart to failure; it is always available in standard gameplay.

Hint refund behavior remains unresolved because hint-count scope is unresolved.

A confirmation dialog is not required by the source spec and should not be added unless deliberately chosen for the case study.

---

# 21. Win evaluation

Use a pure `WinEvaluator`.

A level is won when all of the following are true:

1. every authored slot has a player piece assigned;
2. every authored player piece is placed exactly once;
3. every locked cell is satisfied by any occupying piece(s);
4. every horizontal/vertical player intersection agrees.

Do not require a canonical piece-to-slot assignment if another complete constraint-valid arrangement exists.

The solution-cell map exists for Hint and authored validation; it should not be the only win check.

Because the placement resolver evicts contradictory pieces, a fully occupied board should normally already be intersection-consistent, but `WinEvaluator` should verify consistency defensively.

---

# 22. Win flow

On first transition from unsolved -> solved:

1. stop accepting piece input;
2. save the solved state;
3. grant Monthly Pass level-completion progress exactly once;
4. advance progression to the next level index if one exists;
5. save progression/pass state;
6. show `WinOverlay`;
7. start confetti;
8. leave the completed board visible.

WinOverlay buttons:

- `Main` -> load Main scene.
- `New Game` -> load next progression level when one exists.

Behavior after the final authored level is Unknown in `GAME_SPEC.md`; isolate it in `ProgressionService.HandleNoNextLevel()` or equivalent. Do not silently invent endless replay or loop back to Level 1.

---

# 23. Progression model

Use a single integer progression index.

```text
currentLevelIndex
```

Rules:

- first install -> index 0;
- no level select;
- completed previous levels are not exposed for replay;
- Main/New Game loads the current uncompleted level;
- unfinished state creates Continue/Restart choice;
- completing a level advances to the next configured index.

No stars, scores, timers, grades, or unlock map.

---

# 24. Monthly Pass architecture

The pass is deliberately separate from base gameplay logic.

## 24.1 `MonthlyPassService`

Responsibilities:

```text
bool IsSeasonActive()
TimeSpan GetTimeRemaining()
int GetProgress()
bool IsPremiumUnlocked()
void GrantLevelCompletion(string levelId)
bool CanClaimFree(string milestoneId)
bool CanClaimPremium(string milestoneId)
void ClaimFree(string milestoneId)
void ClaimPremium(string milestoneId)
void MockUnlockPremium()
```

## 24.2 Season timing

Use UTC time.

```text
seasonStart = config.startUtcTicks
duration = 30 days
seasonEnd = seasonStart + duration
```

Do not derive pass time from frame/runtime duration.

The source spec does not define automatic rollover/reset behavior. For the small case study, configure one explicit 30-day season and do not auto-create a new season. If `seasonId` changes in a future config, initialize a fresh `MonthlyPassSave` for that season.

This avoids inventing a recurring live-ops backend.

## 24.3 Progress earning

Only a newly completed progression level grants pass progress.

```text
progress += MonthlyPassConfig.progressPerCompletedLevel
```

Use `rewardedLevelIds` to make this idempotent.

The exact progress amount is authored config because `GAME_SPEC.md` does not define it.

## 24.4 Claimability

Free reward can be claimed when:

- milestone threshold reached;
- not already claimed.

Premium reward can be claimed when:

- milestone threshold reached;
- mock premium is unlocked;
- not already claimed.

Claiming persists the milestone ID immediately.

Do not add inventory/currency side effects unless a later design explicitly defines reward utility.

## 24.5 Mock premium unlock

`MockUnlockPremium()` simply sets:

```text
premiumUnlocked = true
```

and saves.

No IAP package, receipt, store SDK, pricing backend, or platform billing integration.

Retroactive premium claiming is naturally supported because already-reached milestones become claimable after the flag is enabled. The source spec does not explicitly define retroactivity, so keep this behavior isolated to `CanClaimPremium` and document it as a case-study pass design decision if used.

## 24.6 Monthly Pass UI entry

The original game has no supplied Pass UI. For the case-study extension, the simplest access point is a `PassButton` on Main.

This is an explicit extension choice, not reverse-engineered original UI.

---

# 25. Validation/editor safeguards

Create a lightweight `LevelDefinitionValidator` editor utility or `OnValidate()` checks. This is worthwhile even in a small project because level data errors are otherwise hard to diagnose.

Validate:

1. rows/columns > 0;
2. all blocked/locked/solution coordinates are in bounds;
3. no locked cell is blocked;
4. every slot stays in bounds;
5. no slot crosses blocked cells;
6. piece IDs are unique;
7. slot IDs are unique;
8. every piece digit string length matches at least one slot length;
9. solution map has a digit for every active slot cell;
10. initial locked digit equals solution digit at the same coordinate;
11. authored intended solution satisfies every slot/intersection if canonical assignments are provided during authoring.
12. every maximal non-blocked horizontal/vertical run of length 3–7 corresponds to exactly one authored slot;
13. no accidental 2-cell run or >7-cell run exists in Levels 4–6;
14. canonical piece count equals slot count for each level.

Do not build a full custom level editor unless needed. ScriptableObject Inspector authoring is enough for six levels.

---

# 26. Level assets

Author all six levels exactly from `GAME_SPEC.md`. Do not procedurally generate them at runtime.

## 26.1 `Level_01.asset`

- 5×5
- tutorial
- toolbar hidden
- pieces: `12345`, `54321`
- one horizontal 5-cell slot
- one vertical 5-cell slot
- initial locked digits exactly as documented

## 26.2 `Level_02.asset`

- 6×6
- standard toolbar
- pieces: `301`, `792`, `309165`, `357040`, `429156`, `734748`
- slots and locked cells exactly as documented

## 26.3 `Level_03.asset`

- 8×8
- standard toolbar
- source alias: previously documented as `LaterPuzzle_A`
- pieces: `239`, `892`, `1002`, `6643`, `13949`, `90409`, `105522`, `867122`, `2103302`, `2329390`
- topology/slots exactly as documented
- inferred solved assignment may be used to author the solution digit map, but keep the source comment noting that the final assignment was inferred rather than shown in a completed screenshot

## 26.4 `Level_04.asset`

- 9×9
- standard toolbar
- case-study-authored extension
- 12 slots / 18 intersections
- 10 initial locked blue clues
- all lengths 3–7 represented
- exact grid, pieces, slots, canonical assignment, and solution map from `GAME_SPEC.md`

## 26.5 `Level_05.asset`

- 10×10
- standard toolbar
- case-study-authored extension
- 15 slots / 24 intersections
- 9 initial locked blue clues
- six 5-digit candidates and five 7-digit candidates
- exact grid, pieces, slots, canonical assignment, and solution map from `GAME_SPEC.md`

## 26.6 `Level_06.asset`

- 11×11
- standard toolbar
- case-study-authored extension
- 19 slots / 32 intersections
- 8 initial locked blue clues
- eight 7-digit candidates
- exact grid, pieces, slots, canonical assignment, and solution map from `GAME_SPEC.md`

## 26.7 Difficulty rule for authored extension

Do not add mechanics between Levels 4–6. The only intended difficulty increase is data-level complexity:

```text
Level 4: 12 slots, 18 intersections, 10 clues
Level 5: 15 slots, 24 intersections,  9 clues
Level 6: 19 slots, 32 intersections,  8 clues
```

All authored extension numbers remain 3–7 digits. The layouts are static authored content, not runtime procedural generation.

---

# 27. UI state rendering precedence

Transient board rendering should be computed, not permanently written into cell colors.

Recommended precedence for clearly specified states:

```text
ContradictionRed
CompatibleGreen
ReplacementOrange
CandidateOrange               // active piece selected
DigitClassOrange              // no active piece; class/space inspection
LockedBlue / PlayerWhite / Empty
Blocked
```

However, `GAME_SPEC.md` explicitly marks the combined "occupied replacement + crossing contradiction" color precedence Unknown. Keep that one combination in a single method such as:

```text
CellVisualState ResolvePreviewVisualState(...)
```

so it can be changed without changing gameplay rules.

Do not create shaders for glow. For this case-study, tinted cell backgrounds plus a soft overlay Image are sufficient.

---

# 28. Animation implementation

Exact timings are Unknown, so use restrained built-in UI animation only.

Suggested simple implementation:

- piece follows pointer directly or with minimal lerp;
- snap placement: 0.1–0.2 second coroutine lerp;
- orientation change: immediate layout rebuild or short 90-degree visual tween;
- glow: static tinted background/overlay, not pulsing unless desired;
- eviction: immediately return piece to bank or use a short fade;
- win: enable confetti ParticleSystem and fade in WinOverlay.

Do not add an animation framework solely for these effects.

The exact values are implementation tuning, not reverse-engineered facts.

---

# 29. Audio and vibration

The Settings UI must expose Sound and Vibration toggles.

Because exact feedback events are Unknown:

- implement an `AudioSource`/`AudioManager` only if assets are supplied;
- gate all sound playback behind `settings.soundEnabled`;
- gate haptic calls behind `settings.vibrationEnabled`;
- do not hardcode a large feedback matrix.

For WebGL, vibration should safely no-op.

---

# 30. WebGL considerations

## 30.1 Persistence

Use PlayerPrefs JSON and call `PlayerPrefs.Save()` immediately after important state changes.

In WebGL, PlayerPrefs ultimately depends on browser storage and can be cleared by the user/browser. Do not promise cloud persistence.

Do not use direct filesystem paths for save data.

## 30.2 Input

Unity UI pointer interfaces support mouse and touch-style browser input well enough for this game.

Keep drag hit-testing in Canvas coordinates and avoid physics colliders.

## 30.3 Haptics

Treat vibration as unsupported/no-op in WebGL.

## 30.4 External links

If a Privacy Policy URL is later supplied, use `Application.OpenURL` from a direct user click. Do not invent a URL.

## 30.5 Performance

The observed boards are tiny. Instantiate all board cells and bank pieces normally; object pooling is unnecessary.

Avoid per-frame allocations during drag where easy, but do not build custom pooling/collections solely for micro-optimization.

## 30.6 Build settings

Recommended:

- WebGL target
- IL2CPP default WebGL backend
- managed stripping at normal/default level
- compression selected to match hosting support
- portrait-responsive canvas rather than forcing browser orientation

Test mouse drag and touch emulation in the browser build, not only in Editor.

---

# 31. Minimal class dependency map

```text
GameApp
  -> GameConfig
  -> SaveService
  -> ProgressionService
  -> MonthlyPassService

MainScreenController
  -> GameApp.ProgressionService
  -> SettingsPanelController
  -> MonthlyPassPanelController

GameplayController
  -> GameApp.ProgressionService
  -> GameApp.SaveService
  -> GameApp.MonthlyPassService
  -> BoardView
  -> BankView
  -> PlacementResolver
  -> HintService
  -> WinEvaluator

PieceView
  -> GameplayController

BoardView
  -> GridCellView

MonthlyPassPanelController
  -> GameApp.MonthlyPassService
```

No cyclic dependency is needed.

---

# 32. Suggested source files

A compact implementation can stay around the following set:

```text
Core/
  GameApp.cs
  SaveService.cs
  ProgressionService.cs
  SceneNavigator.cs
  SafeAreaFitter.cs

Data/
  GameConfig.cs
  LevelDefinition.cs
  MonthlyPassConfig.cs
  SaveDataV1.cs
  RuntimeModels.cs

Main/
  MainScreenController.cs
  SettingsPanelController.cs

Gameplay/
  GameplayController.cs
  BoardView.cs
  GridCellView.cs
  BankView.cs
  PieceView.cs
  PlacementResolver.cs
  HintService.cs
  WinEvaluator.cs

MonthlyPass/
  MonthlyPassService.cs
  MonthlyPassPanelController.cs
  PassMilestoneView.cs
```

Approximately 18 small scripts is enough. Do not split every DTO or enum into its own file unless desired for readability.

---

# 33. Implementation order for an AI coding agent

Build in this order to reduce rework:

1. Create ScriptableObject data types and author Level 1.
2. Build BoardView and neutral grid rendering.
3. Build bank grouping plus digit-class card press handling.
4. Implement board-cell digit-class focus, including vertical-first intersection cycling.
5. Build PieceView selection/drag and make piece selection override class focus.
6. Implement candidate-slot detection and orange highlights.
7. Implement compatible green preview and auto orientation.
8. Implement player-vs-player mismatch detection and red preview.
9. Implement commit, same-slot replacement, crossing-piece eviction, and bank return.
10. Implement moving already placed pieces.
11. Implement Hint locked-cell creation.
12. Implement WinEvaluator and WinOverlay.
13. Implement save/load, Continue/Restart, and Restart.
14. Author Levels 2 and 3 and validate all resolver behaviors.
15. Author the exact static Level 4, Level 5, and Level 6 data from `GAME_SPEC.md`.
16. Run `LevelDefinitionValidator` across all six assets and verify canonical assignments.
17. Add Main/Settings polish.
18. Add Monthly Pass service/UI/persistence/mock premium.
19. Build and test WebGL.

Do not block the whole implementation on visual polish or exact animation timing.

---

# 34. Test checklist

## 34.1 Core board

- blocked cells render navy;
- authored locked cells render light blue;
- player-filled cells render white;
- shared locked intersection remains blue when matched by player piece.

## 34.2 Selection/preview

- pressing a digit-class card highlights that card and every same-length board slot orange;
- pressing a non-crossing slot highlights all slots of that slot length and the matching bank class;
- first press on an intersection chooses the vertical slot/class;
- repeated presses on the same intersection alternate vertical/horizontal focus;
- pressing a different intersection restarts with vertical priority;
- digit-class focus never selects/moves a piece and never shows green/red validation;
- beginning a piece drag supersedes digit-class focus;

- selecting a piece highlights every same-length slot orange;
- piece automatically adopts horizontal/vertical layout based on targeted slot;
- compatible target is green;
- crossing player mismatch is red;
- occupied same slot shows replacement orange.

## 34.3 Commit

- compatible placement stays;
- target-slot occupant is evicted when replaced;
- contradictory crossing player piece is evicted when incoming piece commits;
- multiple contradictory crossing pieces all return to bank;
- matching crossing pieces remain;
- no player piece exists both on board and in bank.

## 34.4 Movement

- placed piece can be picked up;
- its old contribution disappears during move preview;
- it can be placed in another same-length slot;
- cancellation restores the origin placement;
- dedicated removal behavior remains aligned with the unresolved UX decision.

## 34.5 Hint

- fills exactly one empty active cell;
- uses solution digit;
- becomes light blue;
- remains locked when surrounding pieces move;
- state persists after reload.

## 34.6 Restart/save

- placement persists after scene/app reload;
- hinted cells persist;
- Restart removes player placements and hint-created cells;
- initial authored blue cells remain;
- Continue restores exact saved assignments;
- completed earlier levels are not selectable/replayable.

## 34.7 Win

- only triggers when all slots/pieces are complete and consistent;
- win overlay preserves completed board;
- confetti appears;
- pass progress grants exactly once;
- Main works;
- New Game advances when a next level exists.

## 34.8 Monthly Pass

- 30-day configured season timing uses UTC;
- progress persists;
- level completion adds configured progress once;
- free reward claims persist;
- premium claims are locked until mock unlock;
- mock premium state persists;
- no real IAP package is present.

---

# 35. Explicit non-goals

Do not implement any of the following unless the project brief changes:

- arithmetic equations;
- procedural puzzle generation;
- level editor tooling beyond simple validation;
- replayable level select;
- lives/fail states;
- score/timer/stars;
- analytics SDK;
- ads;
- real IAP;
- account/login/cloud save;
- server-authoritative Monthly Pass;
- remote config;
- currencies/inventory solely to make rewards look more complex;
- undo stack;
- hint solver beyond reading the authored solution cell;
- animation framework;
- object pooling;
- dependency-injection framework.

---

# 36. Remaining implementation decisions that must stay visible

These are not engineering omissions; they are Unknown in `GAME_SPEC.md` and should remain explicit until clarified or deliberately chosen as case-study behavior.

## 36.1 Locked-blue contradiction

`PlacementResolver` must detect it and return `LockedConflictUnknown`. A final commit policy must not be claimed as original behavior without clarification.

## 36.2 Piece removal gesture

Moving/repositioning is defined; intentional removal-to-bank gesture is not. Keep it in one drop-resolution method.

## 36.3 Hint count semantics

UI may show the documented count badge, but storage scope, decrement/refund rules, and replenishment are not source-defined.

## 36.4 Continue/Restart visual presentation

Behavior is defined. This implementation uses a Main-scene modal because it is minimal; label it as a case-study UI choice.

## 36.5 End of authored content

Level 6 is the final authored level. Do not automatically replay earlier levels. Final-level New Game behavior requires a deliberate project decision.

---

# 37. Definition of done

The case-study implementation is ready when:

- it builds in Unity 2022 LTS+;
- a WebGL build is playable;
- all source-defined grid, placement, replacement, mismatch/eviction, hint, restart, persistence, progression, settings, and win behavior works;
- all six defined levels are playable, with Levels 4–6 clearly identified as case-study-authored content;
- the Monthly Pass has a 30-day configured season, free/premium lanes, level-completion progress, claim states, persistence, and a mock premium unlock;
- no arithmetic mechanic has been introduced;
- no Unknown source behavior is silently represented as verified original behavior.
