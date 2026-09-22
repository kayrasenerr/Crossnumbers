# Crossnumbers — Visual Design Specification

**Target:** Unity 2022 LTS+ / portrait WebGL case-study build  
**Visual evidence:** supplied screenshots in the project sources  
**Companion files:** `GAME_SPEC.md`, `IMPLEMENTATION_SPEC.md`, `level_data.md`, `monthly_pass.md`

This file is the implementation source of truth for colors, typography, line weights, spacing, component styling, and responsive sizing.

The screenshots provide strong visual evidence but are JPEG captures rather than original design assets. Therefore:

- **Observed** = directly visible in screenshots.
- **Measured approximation** = sampled/estimated from screenshot pixels and geometry.
- **Implementation choice** = chosen to reproduce the observed style consistently where exact source values cannot be recovered.

Do not treat approximate hex values or font identification as original source metadata.

---

# 1. Overall visual language

**Observed**

Crossnumbers uses a clean, soft casual-puzzle style:

- portrait layout;
- pale blue/gray backgrounds;
- dark navy grid and text;
- large rounded UI cards;
- minimal iconography;
- strong semantic state colors;
- almost no decorative chrome during gameplay;
- bright celebratory win screen;
- generous spacing and centered composition.

The interface should feel flat and readable rather than glossy or skeuomorphic.

---

# 2. Reference canvas and scaling

All supplied screenshots are `945 × 2048` portrait images.

Use this as the design reference resolution:

```text
Canvas Scaler: Scale With Screen Size
Reference Resolution: 945 × 2048
Screen Match Mode: Match Width Or Height
Match: 0.5
```

Use anchors so the UI remains usable on other portrait aspect ratios.

Recommended horizontal safe margin:

```text
~5% of screen width minimum
```

Gameplay board may expand farther than ordinary cards on large grids.

---

# 3. Core color tokens

Values below are measured/estimated from the screenshots.

| Token | Hex | Status | Use |
|---|---|---|---|
| `GameplayBackground` | `#DAE4F0` | measured approximation | gameplay/tutorial background |
| `MainBackgroundLight` | `#EDF4FC` | measured approximation | light portion of main background |
| `MainBackgroundBlue` | `#DCECFD` | measured approximation | blue portion of main background |
| `SettingsBackground` | `#EAEDF2` | measured approximation | settings background |
| `GridBlocked` | `#28394D` | strongly measured | blocked cells |
| `GridStroke` | `#101820` | measured approximation | cell lines / board outline |
| `PrimaryText` | `#34445C` | measured approximation | digits and normal dark text |
| `LockedBlue` | `#C9DAEC` | strongly measured | game-supplied locked cells |
| `PlayerWhite` | `#FFFFFF` | observed | player-filled / empty active cells |
| `CandidateOrange` | `#FFE6A3` | strongly measured | same-length slot highlight |
| `ClassHeaderOrange` | `#FFDB83` | strongly measured | selected digit-class header |
| `ClassBodyOrange` | `#FFF8E5` | strongly measured | selected digit-class body |
| `ValidSlotGreen` | `#BCE9C0` | measured approximation | compatible target slot |
| `ValidPieceGreen` | `#9FE7A5` | strongly measured | dragged/preview number cells |
| `ConflictRed` | `#FFD1D4` | strongly measured | contradiction strip/cells |
| `BankHeaderSlate` | `#7D8FA5` | strongly measured | normal digit-class header |
| `DisabledNumber` | `#E9E9E9` | measured approximation | unavailable/placed bank number |
| `PrimaryBlue` | `#1F82FC` | strongly measured | New Game, toggles, emphasis |
| `ToolbarCircle` | `#D0DEF0` | measured approximation | back/restart/hint circular background |
| `ToolbarIconBlue` | `#3166AF` | measured approximation | toolbar icon strokes |
| `PremiumGold` | `#E6B84F` | implementation choice | Monthly Pass Premium accent only |
| `PremiumPaleGold` | `#FFF3D3` | implementation choice | Premium reward backgrounds |

## 3.1 Semantic-color rule

The following colors have gameplay meaning and must not be replaced by cosmetics:

```text
Locked blue   = game-owned immutable digit
White         = normal active/player cell
Orange        = digit-class/candidate/replacement focus
Green         = compatible placement
Red/pink      = contradiction
Dark navy     = blocked cell
```

Monthly Pass cosmetics may affect outer frames/confetti only.

---

# 4. Grid construction

## 4.1 Cell sizing

The 5×5 and 6×6 screenshots use cells of roughly the same physical size; the 8×8 board scales down to remain within the screen.

Recommended formula at the 945px reference width:

```text
maxBoardWidth = 0.88 × canvasWidth
preferredCellSize = 130 px
cellSize = min(preferredCellSize, maxBoardWidth / max(rows, columns))
```

Center the board horizontally.

This produces approximately:

```text
5×5 -> 650 px board
6×6 -> 780 px board
8×8 -> 830 px board
9–11 grids -> shrink to fit same max width
```

For authored Levels 9×9 through 11×11, preserve readability by using the same maximum board width rather than scrolling the grid.

## 4.2 Grid strokes

**Measured from screenshots**

At the `945px` reference width:

```text
internal cell line: 4–5 px
outer board border: 7–9 px
```

Implementation target:

```text
InternalStroke = 4 px
OuterStroke = 8 px
```

Scale with CanvasScaler.

Use crisp solid strokes; do not blur the grid lines.

## 4.3 Outer corners

Observed board outline has slightly rounded corners while internal cells remain square.

Recommended:

```text
Board outer corner radius: ~10 px at reference resolution
Cell corner radius: 0
```

Do not round individual grid cells.

## 4.4 Normal cell states

```text
Blocked:      GridBlocked
Empty active: PlayerWhite
Player digit: PlayerWhite
Locked digit: LockedBlue
```

Digit color remains `PrimaryText` in all normal active states.

## 4.5 Candidate / class-focus state

Both piece selection and the new digit-class/space inspection mechanic use the same orange family.

```text
Board slot fill: CandidateOrange
Digit-class header: ClassHeaderOrange
Digit-class body: ClassBodyOrange
Digit-class text: warm dark brown/navy, approx #9A693F
```

Do not add a second color for class focus. The shared orange language is intentional.

## 4.6 Compatible placement

The screenshot shows a stronger green on the dragged number and a lighter green on the board slot beneath it.

Use:

```text
Target slot: ValidSlotGreen
Dragged piece: ValidPieceGreen
```

Keep grid strokes dark and visible through this state.

## 4.7 Contradiction

Use `ConflictRed` over the contradictory strip/cells.

The selected piece may still contain green-compatible cells while the contradicting crossing digit/strip is red, matching the screenshot.

Do not use warning icons or error text.

---

# 5. Board digits

## 5.1 Typeface

**Observed:** rounded/clean geometric sans-serif; exact source font cannot be determined from screenshots.

Preferred visual match:

```text
Nunito / Nunito Sans / similar rounded geometric sans
```

For a zero-dependency Unity build, TextMeshPro's bundled font is acceptable if no matching font asset is already available. Do not block implementation on exact font identification.

## 5.2 Board digit weight

Use a medium-to-semi-bold weight.

Digits should be highly legible but not as heavy as the `Congratulations!` heading.

## 5.3 Board digit size

Scale from cell size:

```text
fontSize ≈ 0.62 × cellSize
```

Clamp if necessary:

```text
minimum ~42 px
maximum ~82 px
```

Center horizontally and vertically.

---

# 6. Number-bank / digit-class cards

## 6.1 Card layout

Normal group card:

```text
┌────────────────────┐
│      6 digits      │  <- slate header
├────────────────────┤
│ 309165    734748   │  <- white body
└────────────────────┘
```

Selected class:

```text
┌────────────────────┐
│      6 digits      │  <- yellow/orange header
├────────────────────┤
│ numbers...         │  <- pale yellow body
└────────────────────┘
```

The **entire card**, especially the header region, should be pressable for digit-class focus.

## 6.2 Dimensions

Recommended at reference resolution:

```text
corner radius: 10–14 px
header height: 70–80 px
body minimum height: 100–120 px
horizontal card padding: 18–24 px
vertical gap between cards: 22–28 px
column gap: 22–28 px
```

The screenshot shows:

- one full-width card when only one/two classes are present;
- two-column cards when many digit classes exist;
- the final odd group may span the full width.

Recommended bank width:

```text
~86–88% of canvas width
```

## 6.3 Normal card colors

```text
Header: BankHeaderSlate
Header text: white
Body: white
Number text: PrimaryText
Unavailable/placed number text: DisabledNumber
```

## 6.4 Selected class card

```text
Header: ClassHeaderOrange
Body: ClassBodyOrange
Optional border: 2–3 px in #F4C75E
```

Use a subtle shadow under the entire card.

## 6.5 Header typography

```text
weight: Bold
reference font size: ~42–48 px
alignment: centered
```

## 6.6 Bank-number typography

```text
weight: Regular/Medium
reference font size: ~44–52 px
tracking: small positive tracking if available
```

Keep numbers centered with generous horizontal spacing.

---

# 7. Digit-class interaction visuals

The new inspection mechanic must visually reuse the existing selected-class style.

## 7.1 Press class card

When `x digits` is pressed:

- that class header becomes `ClassHeaderOrange`;
- its body becomes `ClassBodyOrange`;
- every `x`-cell slot becomes `CandidateOrange`;
- no number piece becomes green/red;
- no number is moved.

## 7.2 Press non-crossing board space

Apply exactly the same class-card and all-slot orange state for that slot length.

## 7.3 Press crossing cell

First press selects the vertical slot's class; second consecutive press selects the horizontal slot's class; continue alternating.

No extra arrow/orientation icon is required. The changed highlighted slot-length set is the feedback.

If both crossing slots are the same length, the visual can remain unchanged while internal orientation focus alternates.

## 7.4 Piece selection overrides inspection

Once a number piece is selected/dragged, use the piece-placement state rendering from `GAME_SPEC.md`; do not layer a second class-focus tint on top.

---

# 8. Toolbar

Standard gameplay screenshots show:

```text
Back       Restart                              Hint + count badge
```

Use circular pale-blue button backgrounds.

Recommended reference values:

```text
circle diameter: 78–88 px
icon size: 42–50 px
left margin: 42–50 px
button gap: 42–50 px
right margin: 42–50 px
```

Colors:

```text
circle: ToolbarCircle
icon: ToolbarIconBlue
badge: PrimaryBlue
badge text: white
```

Hint badge is a small blue circle overlapping the icon's upper-right.

Do not show the standard toolbar in tutorial Level 1 if the game spec says it is hidden there.

---

# 9. Main screen

## 9.1 Background

Observed: pale blue/white vertical gradient with a very faint rotated number-grid decoration.

Use:

```text
top/center: MainBackgroundLight
lower blue: MainBackgroundBlue
```

Decorative grid:

```text
opacity: 4–8%
rotation: approximately -15°
color: pale desaturated blue
```

Do not make the decorative puzzle readable enough to compete with the logo.

## 9.2 Logo

Use the supplied/custom `crossnumbers` logo as an image if available.

If it must be rebuilt as text, use bold dark navy, but an image is preferred because the `o/x` mark is custom.

## 9.3 Settings icon

Place the gear in the upper-right with no heavy card background.

Color:

```text
PrimaryText / ToolbarIconBlue range
```

## 9.4 New Game button

Observed bright-blue rounded pill.

Recommended:

```text
width: ~400 px at reference resolution
height: ~88–96 px
corner radius: 44–48 px
fill: PrimaryBlue
label: white, bold
```

Keep it near the bottom with generous side margin.

---

# 10. Settings screen

Observed style is closer to iOS-style grouped settings than the gameplay screen.

```text
Background: SettingsBackground
Cards: white
Text: PrimaryText
Action/toggle blue: PrimaryBlue
```

Recommended:

```text
page horizontal margin: 55–65 px
card radius: 18–22 px
row height: 72–82 px
section gap: 22–30 px
```

Title:

```text
centered
bold
~30–34 px reference size
```

`Done`:

```text
upper-right
PrimaryBlue
semi-bold
```

Toggle:

- enabled track: `PrimaryBlue`;
- thumb: white;
- use standard rounded toggle proportions.

Chevron rows use a light cool-gray chevron.

---

# 11. Win screen

## 11.1 Background

Observed full-screen saturated blue gradient.

Approximate gradient:

```text
top:    #1C52BA
middle: #1674EA
bottom: #1C44A4
```

## 11.2 Congratulations heading

Observed large yellow/orange text.

Recommended:

```text
weight: ExtraBold/Bold
font size: ~58–66 px
alignment: centered
fill: yellow-orange gradient from #FFD84A to #FFAA3D
```

If text gradient is inconvenient, use solid `#FFC04A`.

## 11.3 Completed board

Render the completed board cleanly over the win background.

Do not show:

- candidate orange;
- compatible green;
- contradiction red;
- digit-class focus.

Use settled grid colors only.

## 11.4 Confetti

Use multicolored small rectangular/curved particles around the board and heading.

Keep particle density moderate so the board stays readable.

## 11.5 Buttons

Primary win button is a white pill:

```text
fill: white
label: dark navy
height: ~86–94 px
corner radius: half height
```

Secondary `Main` action may be simple white text beneath it.

---

# 12. Monthly Pass visual extension

The Monthly Pass is not screenshot-derived, so it should look like a natural extension of the base UI rather than a separate art style.

Reuse:

- `PrimaryText`;
- `PrimaryBlue`;
- white cards;
- `BankHeaderSlate`;
- the same rounded corners;
- the same font family;
- the same shadow softness.

Free lane:

```text
header/accent: BankHeaderSlate / PrimaryBlue
card: white
```

Premium lane:

```text
accent: PremiumGold
pale card: PremiumPaleGold
```

Do not use Premium gold for gameplay candidate slots.

---

# 13. Shadows

Screenshots show restrained card depth.

For rapid Unity implementation, use a standard UI `Shadow` rather than custom blur shaders.

Recommended:

```text
shadow color: rgba(30,45,65,0.20)
offset: (0, -5 px) in visual direction
```

If Unity's UI coordinate convention makes this appear upward, invert the Y offset.

Use shadow on:

- bank cards;
- major white panels;
- selected class cards;
- large buttons only if needed.

Do not shadow individual grid cells.

---

# 14. Animation timings

Exact source timings are not recoverable from still screenshots.

Use restrained implementation timings:

```text
button press scale:       0.08–0.12 s
piece snap:               0.12–0.18 s
class highlight change:   immediate or <=0.10 s
orientation swap:         0.10–0.15 s
reward/card pop:          0.15–0.20 s
win overlay fade:         0.20–0.30 s
```

Do not pulse orange class focus continuously.

---

# 15. Responsive layout rules

1. Keep the board fully visible horizontally.
2. Shrink cell size for Levels 8×8 and larger rather than allowing horizontal scrolling.
3. Keep bank cards below the board in a vertical `ScrollRect` if the screen height cannot fit every class.
4. Keep top toolbar anchored to safe-area top.
5. Preserve at least ~20 px visual gap between board and bank.
6. Do not allow Monthly Pass or settings overlays to modify puzzle board scaling.

For landscape WebGL windows, preserve the portrait game canvas centered with unused side space rather than redesigning the game into a landscape layout.

---

# 16. Recommended Unity visual config

Create one small `GameplayVisualConfig` ScriptableObject or serialized config object:

```text
Color gameplayBackground
Color gridBlocked
Color gridStroke
Color primaryText
Color lockedBlue
Color playerWhite
Color candidateOrange
Color validSlotGreen
Color validPieceGreen
Color conflictRed
Color bankHeaderSlate
Color classHeaderOrange
Color classBodyOrange
Color disabledNumber
Color primaryBlue
Color toolbarCircle
Color toolbarIconBlue

float internalGridStroke = 4
float outerGridStroke = 8
float maxBoardWidthRatio = 0.88
float preferredCellSize = 130
float boardDigitSizeRatio = 0.62
```

Other screen-specific colors can stay serialized on their own views if that is simpler.

Do not create a large theming framework.

---

# 17. State-rendering precedence

When multiple states could affect one cell, render in this order:

```text
1. ContradictionRed
2. CompatibleGreen
3. ReplacementOrange / active-piece CandidateOrange
4. DigitClassOrange
5. LockedBlue / PlayerWhite / Empty
6. Blocked
```

Locked ownership still matters underneath previews. When the preview ends, a locked cell returns to `LockedBlue`.

The unresolved locked-blue contradiction behavior remains governed by `GAME_SPEC.md`; this visual file does not invent its commit rule.

---

# 18. Screenshot-specific implementation checklist

## Gameplay

- [ ] pale blue gameplay background
- [ ] dark navy blocked cells
- [ ] white active/player cells
- [ ] light-blue locked cells
- [ ] 4 px internal grid lines at reference scale
- [ ] 8 px outer board border
- [ ] centered large navy digits
- [ ] slate bank headers
- [ ] orange selected class/candidate slots
- [ ] light/dark green compatible preview
- [ ] pink/red contradiction strip
- [ ] subtle bank-card shadow

## Main

- [ ] pale blue/white gradient
- [ ] faint rotated puzzle decoration
- [ ] centered logo
- [ ] top-right gear
- [ ] large blue pill New Game button

## Settings

- [ ] light gray background
- [ ] white grouped cards
- [ ] blue toggles
- [ ] centered title
- [ ] blue Done action

## Win

- [ ] saturated blue gradient
- [ ] yellow/orange Congratulations heading
- [ ] confetti
- [ ] settled completed board
- [ ] white pill New Game button
- [ ] white Main text action

---

# 19. Authority rule

If implementation documents disagree about visuals:

1. screenshot-observed semantics in `GAME_SPEC.md` win;
2. this `VISUAL_DESIGN_SPEC.md` supplies concrete rendering values;
3. `monthly_pass.md` may add Premium-only gold accents but cannot override gameplay semantic colors;
4. `level_data.md` contains no visual overrides.
