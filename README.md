# Crossnumbers

A from-scratch recreation of the Crossnumbers puzzle game, built in Unity 2022 LTS and compiled to WebGL, with a new Monthly Pass reward feature layered on top. Built as an AI-assisted development case study; the engineering log and full process report are maintained alongside this project and referenced throughout this README.

**▶ [Play it here](https://kayrasenerr.github.io/Crossnumbers/)** — compiled WebGL build hosted directly from this repo, runs in-browser with no install required.

## What this is

Crossnumbers is a grid-based number-placement puzzle: numbers are dragged from a bank onto a crossword-style grid so that every horizontal and vertical run of digits forms a correct equation. This build reproduces that core loop across six playable levels and adds a Monthly Pass — a 30-day, 30-step Free/Premium reward track driven by level completion, with a mocked Premium unlock and no real in-app purchases.

## Requirements

*(Applies if the Unity source is added to this repo later — the hosted build above needs nothing but a browser.)*

- Unity **2022.3.62f3** (or a compatible 2022 LTS patch) with the **WebGL Build Support** module installed
- A valid Unity license activated in the editor (see [Known issues](#known-issues) below if validation fails)

## Opening the project

*(For when the Unity source is added — not needed to play the hosted build.)*

1. Open the project folder in Unity Hub and let it import.
2. Open the **Main** scene under `Assets/Scenes/`. If Unity ever restores an empty or untitled scene on launch, `GameEntryScene` will reopen Main automatically, and `GameBootstrap` will redirect any non-game runtime scene back to Main at runtime.
3. Press Play. The game runs directly from the runtime data model — see [Fresh project behavior](#fresh-project-behavior) if you haven't generated the ScriptableObject assets yet.
4. To rebuild the WebGL output yourself: **File → Build Settings → WebGL → Build**.

## Fresh project behavior

*(Applies once the Unity source is added — irrelevant to the hosted build.)*

The project ships with an editor-side asset builder that generates the persistent ScriptableObject assets and prefabs (level definitions, pass configuration, reward definitions) from the specs in `Docs/`. If those generated assets haven't been created yet in your Unity session, a `RuntimeConfigFactory` fallback keeps the game fully playable regardless — you don't need to run the asset builder before pressing Play, only before making persistent edits of your own to level or pass data.

## Project structure

*(Reflects the full Unity project, for reference once the source is added alongside the build.)*

| Path | Contents |
|---|---|
| `Assets/Scenes/` | `Main` and `Gameplay` scenes |
| `Docs/` | Authoritative specs: `GAME_SPEC`, `IMPLEMENTATION_SPEC`, `LEVEL_DATA`, `MONTHLY_PASS`, `VISUAL_DESIGN_SPEC` (and `UPDATED SourceSpecs` for the revised versions) |
| `Docs/screenshots/` | Reference screenshots of the original game, used during the visual/UX pass |
| `Validation/UI/` | Editor-captured screenshots and smoke-test output from `UiRepairCapture` |
| `AI_DEVELOPMENT_LOG.md` | Dated engineering log: audit notes, decisions, build history, bug fixes |
| `TESTING_CHECKLIST.md` | Manual test steps for anything that can't be verified automatically (editor and WebGL) |

## Rendering pipeline

The project uses Unity's **Built-in render pipeline**, not Universal 2D. The original specs assumed Universal 2D, but the actual project baseline was already on Built-in with no game content yet — switching pipelines wasn't necessary for any specific requirement, so the existing pipeline and packages were preserved rather than migrated.

## Core mechanics

- **Grid-based placement** — numbers are dragged from a scrollable bank onto the puzzle grid, with automatic orientation detection and aligned placement previews.
- **Validation** — placing a number gives per-digit feedback rather than an all-or-nothing result (e.g. dropping `632` onto a target of `_4_` shows green–red–green, since only the middle digit conflicts). Drops that don't land on a matching-length target return to the bank; an invalid move off an occupied slot restores that slot's prior content.
- **Locked clues** — numbers pre-filled by the level itself are rendered as fixed, non-interactive blue digits: excluded from the bank, from placement, and from win evaluation, and any drop onto a locked cell is rejected without mutating state.
- **Six playable levels**, authored exactly as specified in `LEVEL_DATA`. After Level 6, the New Game controls become an inactive, locked "Level 7 Coming Soon" placeholder rather than pointing at content that doesn't exist.
- **Continue / Restart** — a level with saved progress shows a Continue button above a smaller Restart option; a fresh level shows only New Game.
- **Hints** — single-cell hints drawn from a shared balance that also feeds the Monthly Pass.
- **Level 1 tutorial** — a short guided sequence highlighting the first horizontal and then vertical placement with an animated hand, dimmed background, and cell highlighting.

## Monthly Pass

A 30-day, 30-step reward track with separate Free and Premium lanes, earning progress only from first-time level completion (5 keys per level; six levels reach exactly 30/30). Premium can be unlocked at any point via a mocked, iOS-style purchase confirmation modal — no real transaction occurs — and unlocking retroactively opens any Premium rewards already reached, without granting extra progress.

- **Free track:** 17 Hint charges + a silver completion emblem
- **Premium track:** 19 additional Hint charges + cosmetics (gold bank frame, star confetti, sapphire board frame, gold completion emblem) — **36 Hint charges combined**
- All progress, claims, and Premium ownership persist across sessions, including a browser refresh, backed by immediate PlayerPrefs-based JSON saves.
- The season expires 30 days after first initialization; any reached-but-unclaimed rewards are settled automatically once, after which the track becomes a read-only "Ended" view.

Full behavioral specification lives in `Docs/MONTHLY_PASS.md` once the Unity source is added alongside the build.

## Known issues

- Running Unity in batch mode for automated validation can fail if the local Unity Licensing Client can't establish its IPC channel (Unity exits with code 199 before compiling). If this happens once the source is added, open the project normally in the editor — compilation and Play mode work correctly under an interactive, licensed session — and fall back to the manual steps in `TESTING_CHECKLIST.md`.
- The Monthly Pass season clock is client-side and authoritative for this build; changing the local system clock will affect season timing. Acceptable for this case study, which demonstrates the system rather than defending against a determined player.

## Verification

*(Applies to the Unity source, once added — the hosted build has no editor tooling.)*

An opt-in editor capture tool (`UiRepairCapture`) is included for internal verification only. It saves to a separate `crossnumbers.ui-audit` namespace (so it never touches real player saves), captures Main, Settings, the Pass panel, all six levels, and several specific UI states, and runs a small smoke test that places the two tutorial pieces and confirms the resulting save and progression update. This is a narrow internal check, **not** a substitute for the manual acceptance testing in `TESTING_CHECKLIST.md` or an actual WebGL browser test — both should still be run before considering a change verified.
