# AI development log

## Audit and implementation plan — 2026-09-21

Read all four supplied specifications before creating project files. Inspected Assets, Packages and ProjectSettings; ignored generated project/cache content. Unity 2022.3.62f3 is installed with WebGL support. This folder is not under Git. Baseline contains only SampleScene and no game scripts or tests.

The installed rendering configuration is Built-in, despite the request describing Universal 2D. Preserve the actual pipeline and packages.

### Requirements and dependency map

| Requirement | System | Depends on |
|---|---|---|
| Six exact authored puzzles, digit strings, static validation | LevelDefinition / editor authoring | LEVEL_DATA |
| Placement, shared digits, replacement, all crossing evictions | PlacementResolver / board state | level data |
| Single-cell hints, locked ownership, constraint-based win | HintService / WinEvaluator | board state, hint balance |
| Drag, pickup, automatic orientation, previews and bank | BoardView / BankView / GameplayController | resolver |
| Continue/restart, linear progression, once-only completion | SaveService / ProgressionService | valid board |
| Thirty steps, two lanes, claims, premium, expiration, cosmetics | MonthlyPassService / panel | config, shared save |
| Main/settings/win/navigation, responsive portrait UI | scene controllers and UI prefabs | services |
| Browser persistence and deployment | immediate PlayerPrefs JSON, WebGL build | all systems |

Implementation follows that dependency order. No arithmetic, replay/level-select, real payments, or new currencies.

### Document conflicts raised before authoring

- GAME_SPEC Level 3 has a locked 3 at zero-based (2,1); LEVEL_DATA explicitly leaves that cell empty and counts 13 clues.
- The Level 3 pictured eviction example intersects a V4 digit marked game-locked in the supplied grids. Locked ownership and player eviction cannot both describe that same digit.
- MONTHLY_PASS exact table totals 17 Free + 19 Premium hint charges = 36; its summary says 34 combined.
- MONTHLY_PASS explicitly supersedes earlier placeholder pass decisions: rolling persisted season, functional hints/cosmetics, shared initial balance 5 and no restart refund.
- User clarification requested for the first three conflicts and remaining locked-drop/removal/end-of-content choices. Unaffected work continues.

Human validation: reference screenshots are named in the specs but not present in the project; visual work can match written descriptions, not claim pixel parity.

### Clarified design choices — 2026-09-21

User chose LEVEL_DATA exactly for Level 3, including its 13 initial clues; player eviction examples will use other unlocked intersections. Locked-blue conflicts are rejected without mutating state. Off-board drop returns a piece to the bank, Escape cancels an active move, and Continue/Restart is a simple modal. After Level 6, New Game controls become an inactive locked “Level 7 Coming Soon” mock.

The exact 30-step Monthly Pass reward table is authoritative: it contains 17 Free hint charges and 19 Premium hint charges (36 combined), despite an inconsistent 34-charge summary elsewhere in the pass document.

### Initial playable build — 2026-09-21

Created the runtime data model, six authored levels, resolver, hints, win checks, PlayerPrefs JSON saves, linear progression, settings, pass service, dynamic portrait UI, Main/Gameplay scenes, and editor asset builder. Added `RuntimeConfigFactory` so a fresh project remains playable even when generated ScriptableObject assets/prefab have not yet been created. The editor builder can generate persistent assets and prefab in a licensed Unity session.

Validation attempted with Unity 2022.3.62f3 batch mode. The local Unity LicensingClient could not establish its IPC channel and Unity exited with code 199 before compilation. Manual Unity/WebGL validation remains listed in `TESTING_CHECKLIST.md`.

### Safe Mode compile fix — 2026-09-21

Unity reported duplicate `UnityEngine.TestRunner` and `UnityEditor.TestRunner` references in `Assets/Tests/EditMode/Crossnumbers.EditModeTests.asmdef`. The assembly explicitly listed both runners while also requesting the `TestAssemblies` optional reference set, which supplies those runners in Unity 2022.3. Removed the explicit runner entries and retained `TestAssemblies` plus the runtime `Crossnumbers` reference.

### Visual and pointer-input repair — 2026-09-21

Read the new `Docs/UPDATED SourceSpecs` documents and compared their additions with the original specifications. The reference images were located in the sibling `screenshots` folder; the earlier statement that screenshots were unavailable is superseded.

Root causes: changing anchors after assigning stretched RectTransform sizes expanded labels; fixed screen positions were not contained in a portrait frame; bank/pass layout groups had no reliable sizing or scrolling; the gameplay UI only implemented clicks and had no drag lifecycle.

Rebuilt the runtime presentation with a safe-area portrait frame, explicit board geometry, shared visual tokens, grouped scrollable number cards, dimmed placed numbers, a scrollable 30-step pass, working toggle thumbs, drawn toolbar/lock icons, gradients and rounded controls. Added bank/board pointer dragging, automatic orientation, aligned placement previews, blue-conflict rejection, player eviction through the existing resolver, offboard bank returns, and cancellation restoring the origin. Class inspection follows vertical-first repeated crossing-cell cycling. Button presses, orientation changes, placement snaps, screen fades, completed-board settling and win confetti now animate.

Unity compilation succeeded after allowing the installed editor to use the host licensing service. An opt-in editor capture (`UiRepairCapture`) uses a separate `crossnumbers.ui-audit` save namespace. Captures cover Main, Settings, Pass, all six levels, the bottom of the Level 6 bank, class focus, drag preview and the completed board. Its pointer-event smoke check places both tutorial pieces and verifies the saved placement and progression advance. This is a focused smoke check, not the user's full acceptance testing or a WebGL browser test. Generated evidence is under `Validation/UI`; compile/capture logs are at the project root. Player saves and authored level/reward data were preserved.

### Blank Game view startup repair — 2026-09-21

The reported blue, non-interactive view was traced to Unity restoring `Temp/__Backupscenes/0.backup`/an untitled camera-only scene. The runtime UI is created by the Main controller, so that scene had no Canvas or controller to receive input. Added `GameBootstrap` to redirect any non-game runtime scene to `Main`, and `GameEntryScene` to set Main as the editor Play start scene and reopen Main when Unity restores an empty scene. The updated runtime/editor assemblies compile in the open Unity project; no user save data was changed.

### Modal and locked-clue interaction pass — 2026-09-21

Added an iOS-style mock App Store confirmation modal for Premium unlock, plus short Help and Privacy Policy modals. Premium state changes only after Confirm. Authored blue digits now have no pointer handlers, fully authored blue slots are removed from the bank and excluded from placement, and win evaluation treats them as fixed content. Bank numbers start only from a real drag; board drops that miss the matching-length map return to the bank, while an invalid move from an occupied slot restores its origin. Pressing an unrelated board cell clears class/slot focus. The runtime assembly recompiled successfully in the open Unity editor after these changes.


### Updated datasets, tutorial and presentation — 2026-09-22

Imported all six datasets from `Docs/UPDATED SourceSpecs/UPDATED_level_data.md` using `Tools/import_levels.py`. The editor asset builder now uses the same definitions. Level dimensions are 5x5, 6x6, 6x6, 8x8, 8x8 and 9x9. Current-level saves have a stable layout signature, so obsolete boards restart safely without losing overall progression or pass/hint balances. Unchanged legacy Levels 1–2 can resume.

Restored complete authored blue numbers to the bank as dimmed, non-interactive placed entries. Both the board and dragged preview mark individual mismatches pink and all compatible positions green. Different-length spaces switch focus on the first tap; background/blocked spaces clear it. Removed above-board guidance and drop-status text. Invalid releases return pieces to the bank; Escape restores the origin.

Main displays Continue / Level x plus New Game for an unfinished level, and only New Game / Level x for an unseen level. New Game restarts that level directly. Level 7 remains locked and inert. Level 1 has a two-step spotlight guide whose pointing hand automatically travels for one second, holds for two seconds and repeats. Horizontal completion switches to the vertical step; saved progress resumes the correct stage. The tutorial never places pieces itself.

Applied visual_updates.md: licensed Poppins fonts, baked puzzle decoration/shadow/radial background/ribbon sprites, sharper rounded surfaces, double board frame, bank elevation and focus border/glow, larger pills, settings/toggle polish, toolbar shadows, hint badge ring, gradient/outlined win heading, finite multicolor confetti, button motion, placement/return animations and hint pulse. Monthly Pass has navy/gold styling, crown/sparkle decoration, a progress track, milestone rail, reward state cards and fixed actions. Mock purchase confirmation remains required.

Validation uses an isolated Unity copy and a separate audit save namespace. All six datasets passed canonical solvability, static-piece rejection, save round-trip and stale-save checks. A targeted 632 versus _4_ case found exactly one conflict. Runtime captures and pointer checks cover main/continue, settings, pass/purchase, all levels, tutorial stages, previews and win. Full manual acceptance and WebGL browser testing remain for the user. The validation copy reported a Windows long-path warning from the unused Visual Scripting package; compilation and focused captures proceeded.

## Clear all progress setting — 2026-09-22

Added a red Settings action with the requested irreversible-action confirmation, existing Cancel button, and red Clear text. Confirmation replaces gameplay/reward save data with fresh defaults, recreates Monthly Pass/progression services, and loads Level 1 with its tutorial. Audio/vibration preferences survive; mock Premium ownership, claims, cosmetics, hint balance, saved placements, and completed-level rewards reset. Runtime sources compile successfully using Unity 2022.3.62f3 Roslyn and the project assembly references. Interactive confirmation/relaunch verification remains for manual testing.

Clear-progress follow-up: confirming Clear now reloads Main instead of opening Gameplay. The fresh Level 1 is available via New Game.

## Purchase dialog polish — 2026-09-22
Removed blank paragraph breaks, separated the icon/title/body bounds, and reduced the mock-payment note typography. Replaced the letter placeholder with a rounded three-stroke App Store-style mark. Confirmation now persists the mock unlock, refreshes rewards, and plays an 0.85-second opening-lock/pulse animation before dismissing; controls are disabled during it to prevent duplicate confirmation. Runtime Roslyn compilation passed; visual/play-mode verification remains manual.
