# Crossnumbers — Visual Updates

**Target:** Unity 2022 LTS+ / portrait WebGL case-study build
**Purpose:** Patch on top of `VISUAL_DESIGN_SPEC.md` to close the "unpolished" gap between the current build and the reference screenshots (`main_screen.jpg`, `settings.jpg`, `win_screen.jpg`, plus the gameplay screenshots `level*_initial.jpg`).
**Reading order:** Read `VISUAL_DESIGN_SPEC.md` first. This file **adds to and overrides** it wherever the two conflict (see §1).

All values were **measured by sampling pixels** from the `945 × 2048` reference screenshots unless labelled otherwise.

- **Measured** = sampled directly from a screenshot.
- **Observed** = clearly visible, but not reducible to one exact number.
- **Implementation choice** = not recoverable from a still image; chosen to match the look.

---

# 1. What is wrong with the current spec

`VISUAL_DESIGN_SPEC.md` §1 says:

> "The interface should feel flat and readable rather than glossy or skeuomorphic."

That sentence is the root cause of the unpolished look. The screenshots are **not flat**. The board itself is flat, but everything around it is layered: gradients, soft shadows, glows, outlines, and multicolored effects. The current spec also describes several of these as single flat colors.

## 1.1 Corrections to the existing spec

| Item | Old spec says | Screenshots actually show | Section |
|---|---|---|---|
| Overall tone | "flat … rather than glossy" | Flat **board**, polished **chrome** | §2 |
| Gameplay background | single flat `#DAE4F0` | vertical gradient `#EBEFFA` → `#D8E2EE` | §4 |
| Win background | 3-stop vertical gradient | **radial** gradient, bright center, dark corners | §9 |
| `Congratulations!` fill | flat, or `#FFD84A→#FFAA3D` | vertical gradient **plus a brown outline** | §10 |
| Main decoration | "faint rotated number-grid, 4–8%" | perspective-tilted *real puzzle* with blocked cells and digits, fading out | §7 |
| Bank card shadow | `rgba(30,45,65,0.20)`, 5 px | ~16 px falloff, darker directly under the card | §6 |
| Confetti | "small rectangular/curved particles" | 7-colour ribbon capsules with inner shading | §11 |
| New Game button size | ~400 × 88–96 px | **~650 × 144 px** | §8 |
| Bank header height | 70–80 px | **~96 px** | §6 |
| Toolbar circle | 78–88 px | 82 px (back/restart), **97 px** (hint) | §5 |
| Font family | Nunito / Nunito Sans | **Poppins** for all UI text | §3 |
| Settings card radius | 18–22 px | **~40 px** (measured) | §12 |
| Board outline | single ~8 px stroke | double band: navy outer + black inner | §4.3 |
| Win button | none | soft outer glow | §8 |

## 1.2 New design principle (replaces §1 of the old spec)

> **Flat board, polished chrome.**
> Grid cells stay perfectly flat and crisp so the puzzle reads instantly. Every surface *around* the board (backgrounds, cards, buttons, headings, particles) gets depth: a gradient, a soft shadow, a glow, or an outline.

Rules of thumb for every new UI element:

1. Never leave a large surface as a single flat color if the reference shows a gradient.
2. Every elevated surface (card, button) gets a soft, downward-biased shadow or glow.
3. Hero text gets **fill gradient + outline + shadow**, not just a color.
4. Decorative layers stay low-contrast so they never compete with the puzzle.
5. **Do not** add depth to grid cells (see §4.5).

---

# 2. Quick-start checklist

Highest-impact changes first. If time is short, do these in order.

- [ ] **P0** Switch all UI text to **Poppins** (§3)
- [ ] **P0** `Congratulations!` → gradient fill + brown outline + drop shadow (§10.2)
- [ ] **P0** Win background → radial gradient (§9.1)
- [ ] **P0** Confetti → multicolor ribbon particles with shading (§11)
- [ ] **P0** Gameplay background → vertical gradient (§4.1)
- [ ] **P0** Bank cards → real drop shadow (§6.2)
- [ ] **P1** Main screen → perspective puzzle decoration + fade (§7.1)
- [ ] **P1** Blue New Game button → gradient + colored shadow (§8.1)
- [ ] **P1** White win button → outer glow (§8.2)
- [ ] **P1** Button press states (§8.4)
- [ ] **P1** Board outer border → double band (§4.3)
- [ ] **P2** Settings cards → hairline shadow + toggle polish (§12)
- [ ] **P2** Toolbar → soft shadow + press state (§5)
- [ ] **P2** Selected class card → border + glow (§6.4)
- [ ] **P2** Micro-animations (§13)

---

# 3. Typography — switch to Poppins

The old spec recommended *Nunito*. Zoomed screenshots show the screens use **Poppins**: geometric, single-storey `a`/`g` variants in headings, wide round bowls, and heavy weights. This is the single biggest cause of the "off" feel.

**Observed** (identified by letterforms; not confirmed from source metadata).

| Use | Font | Weight | Reference size (px @ 945 wide) | Color |
|---|---|---|---|---|
| `crossnumbers` logo | Poppins (or logo image) | ExtraBold / Bold | ~110 cap-to-ascender | `#34445C` |
| `Congratulations!` | Poppins | **ExtraBold** | cap height ~62 → font ~86–92 | gradient, see §10.2 |
| Button labels (`New Game`) | Poppins | **SemiBold** | ~52–56 | see §8 |
| `Main` text action | Poppins | SemiBold | ~52 | `#FFFFFF` |
| Bank header (`6 digits`) | Poppins | **SemiBold/Bold** | ~46–50 | `#FFFFFF` |
| Bank numbers | Poppins or Noto Sans | Regular/Medium | ~48–52 | `#34445C` |
| Board digits | Poppins | Regular/Medium | ~0.62 × cell | `#34445C` |
| Settings title | Poppins | Bold | ~32–36 | `#152437` |
| Settings rows | Poppins | Medium | ~34 | `#34445C` |
| `Done` | Poppins | SemiBold | ~34 | `#1869DC` |

Notes:

- Poppins is OFL-licensed; importing it as a TextMeshPro font asset is safe.
- Generate the TMP atlas as **SDF** with padding ≥ 6 so outlines and shadows (§10) don't clip.
- Bank numbers in the screenshots use a slightly lighter, more open-tracked face than Poppins; **Poppins Regular with +30–60 tracking** is an acceptable match. Do not block on this.
- Keep board digits **lighter** than the `Congratulations!` heading. Only hero text goes ExtraBold.

---

# 4. Gameplay screen

## 4.1 Background gradient

**Measured** (vertical, sampled down the left edge):

| Y (of 2048) | Color |
|---:|---|
| 0 | `#EBEFFA` |
| 640 | `#E3EAF4` |
| 1280 | `#DFE5F1` |
| 1920 | `#D8E2EE` |

Implementation:

```text
Vertical linear gradient
top:    #EBEFFA
bottom: #D8E2EE
```

This replaces the flat `GameplayBackground = #DAE4F0`. Keep the token, but set it to the bottom stop and add a `GameplayBackgroundTop = #EBEFFA` token.

Optional, subtle (implementation choice): a very faint lighter radial bloom behind the board, `#FFFFFF` at 10–15% alpha, radius ≈ 0.7 × board width. Do not exceed 15%.

## 4.2 Grid cell fills (unchanged; confirmed flat)

**Measured.** These are truly flat, with no gradient inside a cell:

| State | Fill |
|---|---|
| Blocked | `#28394D` |
| Empty / player | `#FFFFFF` |
| Locked | `#C9DAEC` |

## 4.3 Board border — double band

**Measured** (left edge scan at row mid-height): moving inward from the outside, the border is

```text
outer navy band  ≈ 5 px   #152837 – #172737
inner black line ≈ 3–4 px  #000106 – #010409
```

then the first cell. The result is a dark navy outer contour with a near-black inner line, which gives the board its slight "framed" look.

Implementation:

```text
Board outer frame:   Image, color #152837, rounded corner ~10 px, 8–9 px thick
Inner keyline:       Outline, color #000000, 3–4 px, just inside the frame
Internal cell lines: #000000 – #101010, 4 px
```

Do **not** add a shadow, glow, or blur to the board. The screenshots show the board sits directly on the background.

## 4.4 Grid line color

The old spec uses `GridStroke = #101820`. Measured internal lines are darker, effectively **`#000000` to `#101010`**. Use `#0A0F14` (between them) to avoid harsh anti-aliasing on scaled sprites.

## 4.5 What must stay flat

To protect readability:

- ❌ no gradients inside grid cells
- ❌ no inner shadow or bevel on cells
- ❌ no shadow under the board
- ❌ no glow on locked cells
- ❌ no rounded individual cells

Highlights (orange / green / red) are also flat fills. See §6.4 for card-level polish only.

## 4.6 Digit rendering

- Digits are `#34445C` (measured `#2C477E`–`#34445C` range depending on anti-aliasing).
- Use crisp SDF text, no outline, no shadow.
- Locked digits use the same color as player digits; only the cell fill differs.

---

# 5. Toolbar

**Measured:**

| Element | Value |
|---|---|
| Back circle | x 43–124 → **82 px** diameter, center y ≈ 118 |
| Restart circle | x 179–260 → **82 px** diameter |
| Hint circle | x 803–899 → **≈ 97 px** (badge included) |
| Circle fill | `#D2DFF0` (flat) |
| Icon color | `#24468E` – `#2C477E` |
| Hint badge | blue circle, white numeral |

The hint circle reads larger because the badge overlaps its edge. Keep the circle itself at 82 px; the badge protrudes.

## 5.1 Polish to add

```text
Circle fill:        #D2DFF0  (keep flat; do not add a gradient)
Soft shadow:        rgba(49,102,175,0.18), blur 14 px, offset (0, 4 px)
Pressed state:      scale 0.92, fill #C4D5EC, shadow reduced to blur 6 px
Icon stroke:        rounded caps and joins, 5–6 px stroke
Hint badge:         solid #1F82FC, 3 px white ring, tiny shadow rgba(0,0,0,0.20) blur 4
```

Fill is measured; the shadow, press state, and badge ring are an implementation choice to bring the toolbar in line with the softly elevated buttons elsewhere.

---

# 6. Number-bank cards

**Measured card geometry** (3-digit card on level 2):

```text
width:          655 px
header height:  96 px
header fill:    #7D8FA5  (flat in the sampled column)
body fill:      #FFFFFF
```

The old spec's 70–80 px header is too short. Use **96 px** for one/two-class layouts. In two-column layouts (8×8 and up) the card shrinks; scale header proportionally (≈ 0.75×), not to a fixed 70.

## 6.1 Card shape

```text
corner radius:  ~14 px on the outer card
header:         top-rounded only
body:           bottom-rounded only
```

## 6.2 Card shadow (real, downward-biased)

**Measured** shadow falloff below the card (level 2, sampled straight down from the card's bottom edge):

| Distance below | Color | vs. bg `#DDE5F0` |
|---:|---|---|
| 0–3 px | `#8C93A5` | very dark |
| 6 px | `#A2A9B9` | |
| 9 px | `#BAC2CF` | |
| 12 px | `#CED6E3` | |
| 15 px | `#D8E0ED` | |
| 18+ px | `#DDE5F0` | = background |

On the **sides** the shadow is lighter and shorter (~10 px, edge `#B6BAC3`).

Implementation (Unity UI, two `Shadow` components or a shadow sprite):

```text
Shadow A (contact):  rgba(20,30,50,0.35), offset (0, -3), blur ~4
Shadow B (ambient):  rgba(30,45,65,0.22), offset (0, -8), blur ~14
```

Simplest working version with one component:

```text
rgba(30,45,65,0.30), offset (0, -6), blur 14
```

This replaces the old `rgba(30,45,65,0.20)`, which is too faint.

## 6.3 Header polish (implementation choice)

The sampled header is flat, so keep it flat. Add only two refinements:

```text
Top highlight line: 2 px, rgba(255,255,255,0.18) along the top edge of the header
Header text shadow: rgba(40,55,75,0.35), offset (0, -2), blur 2
```

These add a subtle "pressed-in label" feel without changing the measured fill.

## 6.4 Selected (orange) class card

The old spec sets only fill colors. Add:

```text
Header:        #FFDB83  (keep)
Body:          #FFF8E5  (keep)
Border:        3 px #F4C75E  (promote from "optional" to required)
Outer glow:    rgba(255,200,80,0.45), blur 16, no offset
Card shadow:   same as §6.2
Header text:   #9A693F, no shadow
Transition:    ≤ 0.10 s color lerp, no pulse
```

The glow makes "this class is active" visible at a glance even if the board is scrolled off-screen.

## 6.5 Disabled / placed numbers

**Observed:** placed pieces fade to near-invisible `#E9E9E9` on the white body. Keep that. Also fade the digits with a 0.15 s alpha tween instead of snapping.

---

# 7. Main screen

## 7.1 Background — gradient and puzzle decoration

**Measured** vertical gradient (left edge):

| Y | Color |
|---:|---|
| 0–200 | `#EFF6FE` |
| 800 | `#F0F5FB` |
| 1400 | `#E8F0FB` |
| 1800 | `#DCECFC` |
| 2000 | `#D7E9FF` |
| bottom | `#D4EAFF` |

```text
Vertical linear gradient
top:    #EFF6FE
bottom: #D4EAFF   (fully saturated blue toward the very bottom)
```

### The decoration is a real puzzle, tilted in perspective

Contrast-boosting the screenshot shows the background is **an actual crossnumber board**, not a plain grid:

- cells are light, with **darker blocked cells** interspersed;
- **digits** are printed in cells, visible in the upper two-thirds (e.g. `7`, `8`, `5`, `3`, `9`, `2`, `1`, `5`, `8`, `6`, `4`, `5`, `5`, `4`, `9`, `2`, `6`, `7`);
- it is **rotated ≈ −15°** and drawn with **perspective** (cells look larger toward the lower-left; lines converge);
- it **fades to nothing** toward the bottom, where the blue gradient takes over.

Implementation:

```text
Prefab: a large, baked puzzle image (or a Canvas group of ~7×10 cells)
Rotation Z:          -15°
Perspective:         use a 3D-rotated RawImage / Camera-space quad
                     (rotate X ≈ 12–18°, Y ≈ -6–-10°) OR bake into the sprite
Cell fill:           white at 35–45% alpha
Blocked cell fill:   #C9D8EE at 45–55% alpha
Grid lines:          #C4D3EA at 60% alpha, 3 px
Digits:              #C7D0DD at 55–65% alpha, Poppins Medium, ~110 px
Overall layer alpha: 0.85
Bottom fade mask:    alpha 1.0 at y=0 → alpha 0 by y ≈ 1250 (of 2048)
Top-right area:      keep clear enough that the gear icon stays legible
```

Because it is decoration, **bake it to a single sprite** (2048 px tall) rather than building live UI. Nothing here needs to be interactive.

A pure flat "rotated grid" (the old spec) will look noticeably emptier than the reference; the digits and blocked cells are what make it read as *this game*.

## 7.2 Logo

**Observed / measured:** heavy Poppins-style wordmark, color `#34445C`, with the second **`o` replaced by a solid navy circle containing a white `✕`**.

- Use the supplied image if available.
- If rebuilt in code: render the text with the `o` omitted and overlay a filled circle (diameter ≈ x-height × 1.08) with a white `✕` (stroke ≈ 12% of circle diameter, round caps).
- Add a very soft shadow to lift it off the decoration: `rgba(52,68,92,0.18)`, offset (0, -3), blur 8. (Implementation choice; measured logo edges are crisp.)

## 7.3 Settings gear

**Measured color:** `#243954`. Outline-style gear, ~62 px, no background disc. Add:

```text
Pressed:  scale 0.90, alpha 0.7
Idle:     no shadow (icon is line-art; a shadow makes it look heavy)
```

## 7.4 `New Game` button

See §8.1.

---

# 8. Buttons

## 8.1 Primary blue pill (main screen)

**Measured:** x 145–802 → **658 px wide**, y 1573–1716 → **144 px tall** (full pill, radius = 72 px), fill `#1F81FE` → `#1E82FC` (essentially flat in the sample).

The old spec's ~400 × 88–96 px is far smaller than what the screenshot shows. Use the measured size.

```text
Size:           ~658 × 144 px  (≈ 70% of canvas width)
Corner radius:  72 px (full pill)
Fill:           vertical gradient
                  top    #2D8CFF
                  bottom #1A78F0
Top highlight:  1.5 px inner line, rgba(255,255,255,0.35), inset 2 px
Shadow:         rgba(31,130,252,0.35), offset (0, -8), blur 20   <- colored, not grey
Label:          Poppins SemiBold ~54 px, #FFFFFF
Label shadow:   rgba(10,60,140,0.30), offset (0, -2), blur 3
```

Center ≈ (473, 1645). Left/right margin ≈ 145 px.

**Note on the measured shadow:** in the sample, the strip directly under the pill reads only slightly darker than the background. The gradient, colored glow, and label shadow above are chosen so the button reads the same as the reference at rest; keep them subtle. Do not use a black shadow.

## 8.2 White pill (win screen)

**Measured:** x 151–793 → **643 px wide**, y 1575–1718 → **144 px tall**, fill `#FFFFFF`, label color `#2B4568` (darkest glyph pixels `#1F3451`).

**Measured outer glow** (win screen, sampled outward from the pill; bg is `#1D45A5`):

| Distance | Side | Color |
|---:|---|---|
| 1 px | left | `#E5EDFF` (edge anti-aliasing) |
| 4 px | left | `#204BA7` |
| 7 px | left | `#214DAC` (peak) |
| 10 px | left | `#1E49A8` |
| 13 px | left | `#1C47A6` |
| ≥ 16 px | left | `#1B46A5` = bg |
| 4–10 px | below | `#2451AE`–`#2453AF` (peak, stronger) |
| 13–19 px | below | fades to `#1C47A6` |

So it is a **light-blue outer glow, ~16 px wide, a bit stronger below the pill than to the sides**.

```text
Size:           ~643 × 144 px, radius 72
Fill:           #FFFFFF  (flat)
Label:          Poppins SemiBold ~54 px, #2B4568
Outer glow:     rgba(120,170,255,0.35), blur 18, offset (0, -4)
Contact line:   none
```

## 8.3 `Main` text action

**Measured:** pure white `#FFFFFF`, Poppins SemiBold, ~52 px, centered ≈ 100 px below the pill.

```text
Color:      #FFFFFF
Shadow:     rgba(10,30,90,0.35), offset (0, -2), blur 3
Pressed:    alpha 0.7, scale 0.96
```

## 8.4 Button interaction (all buttons)

```text
Press:        scale to 0.96 in 0.08 s, ease-out; shadow shrinks to 60%
Release:      scale back to 1.0 in 0.12 s with a slight overshoot (1.02, then 1.0)
Disabled:     alpha 0.5, remove shadow
Focus / hover (WebGL desktop): brighten fill +6% lightness, 0.10 s
```

These timings extend §14 of the old spec.

---

# 9. Win screen background

## 9.1 Radial gradient

**Measured samples:**

| Location | Color |
|---|---|
| Center (~y 700) | `#1877F5` |
| Left/right of center | `#1A62D1` |
| Top corners | `#1D49AC` |
| Top center | `#1A4EB3` |
| Bottom | `#1C44A4` |
| Lower center (~y 1400) | `#1C51B9` |

So the brightest region is **a soft oval centered slightly above the middle of the screen (≈ y 700–760)**, darkening to a deep blue at every edge. It is **not** a plain top-to-bottom gradient.

```text
Type:          Radial
Center:        (50% x, ~36% y)   (≈ y 740 of 2048)
Radius:        ~65% of screen height, slightly elliptical (wider than tall is fine)
Stops:
  0%   #1877F5
  35%  #1A62D1
  70%  #1B4FB5
  100% #1C44A4
```

Simplest Unity implementation: a full-screen sprite baked from these stops (256 × 512 is plenty), scaled to cover. Avoid banding by adding 2–3% noise or exporting at 8-bit dithered.

## 9.2 Soft spotlight (implementation choice)

To frame the completed board, add a faint radial highlight behind it:

```text
Color:   rgba(255,255,255,0.06)
Radius:  ≈ 0.6 × board width
```

Skip if it reduces confetti contrast.

---

# 10. Congratulations heading

The screenshot heading is a **stacked-effect hero text**. It is the most visible polish gap.

## 10.1 Measured geometry

```text
x-extent:     63 – 884 px   (≈ 822 px wide, ≈ 87% of canvas width)
cap 'C' height: ~62 px
Vertical center: ≈ y 390
Font:         Poppins ExtraBold (see §3)
```

## 10.2 Fill gradient (vertical, top → bottom)

**Measured** (per-row average of fill pixels):

| Y within heading | Color |
|---:|---|
| 362 (top) | `#FCE283` |
| 378 | `#FBDF7C` |
| 394 | `#FAD16C` |
| 410 | `#F8C358` |
| 418–442 (bottom) | `#F8BD4C` → `#F9BB47` |

```text
Fill gradient (vertical)
top:    #FCE283   (pale warm yellow)
middle: #FAD16C
bottom: #F8BD4C   (amber)
```

This is **paler and softer** than the old spec's `#FFD84A → #FFAA3D`, which would look too saturated.

## 10.3 Outline

**Measured:** dark brown-orange, e.g. `#88432A`, `#844540`, `#8D3E17`. Dominant tone ≈ **`#88432A`**. The stroke is roughly **4–5 px** at reference scale, sitting just *outside* the letter edge.

```text
Outline color:  #88432A
Outline width:  4.5 px (TMP outline ≈ 0.16–0.20 in SDF units at ~88 px)
Outline join:   round
```

## 10.4 Shadow / depth

**Observed:** the letters have a slightly lifted look against the blue. Add:

```text
Drop shadow:    rgba(20,40,110,0.45), offset (0, -5), blur 6
Top edge gloss: optional 2 px white line, alpha 0.35, clipped to the upper third of the letters
```

TextMeshPro setup that reproduces this:

```text
Material: TextMeshPro/Distance Field
Face Color:      white      (gradient applied via "Color Gradient" preset)
Color Gradient:  Vertical — Top #FCE283, Bottom #F8BD4C
Outline:         Color #88432A, Thickness 0.18, Softness 0
Underlay:        Color rgba(20,40,110,0.45), OffsetY -0.6, Softness 0.35
```

If TMP gradients are inconvenient, fall back to **solid `#F9D26E`** for the fill; keep the outline and shadow regardless. The outline is what makes it read as "the game's heading".

## 10.5 Entrance animation (implementation choice)

```text
Scale:   0.6 → 1.08 → 1.0 over 0.35 s (ease-out-back)
Alpha:   0 → 1 over 0.2 s
Delay:   0.15 s after the overlay fade begins
Idle:    none (do not pulse)
```

---

# 11. Confetti

## 11.1 What the reference shows

Zoomed inspection shows **~50–60 particles** on screen, in **7 colors**, of **three shapes**:

1. **Straight capsules**, short rounded rectangles.
2. **Curved / "S" ribbons.**
3. **Hooked / "J" ribbons.**

Each has **rounded ends** and a **soft light-to-dark inner shading** (lighter on one edge, deeper on the other), so they read as glossy plastic ribbons, not flat rectangles.

Confetti sits **in front of the background and the heading**, and **behind the board and buttons** in some cases (it is partially hidden under the pill). In the reference, a few pieces overlap the heading and the board edge.

## 11.2 Palette

Approximate (sampled/observed; exact per-pixel values vary with shading):

| Color | Base hex | Notes |
|---|---|---|
| Red / hot pink | `#F0284F` | slightly crimson |
| Magenta | `#F02ED0` | rare, small |
| Green | `#22D05A` | bright, most common |
| Yellow | `#FFEA1F` | bright lemon |
| Orange | `#FF8A1F` | |
| Cyan | `#3EE8F2` | |
| Purple / violet | `#7A3DF0` | |
| Blue | `#2A8BFF` | low contrast on the bg; keep sparse |

Weighted spawn: green 20%, yellow 16%, red 16%, orange 14%, cyan 12%, purple 10%, blue 6%, magenta 6%.

## 11.3 Particle sizing and shading

```text
Length:            22–48 px  (reference resolution)
Thickness:         10–14 px
Corner radius:     full (capsule)
Curved variants:   bend 25–70°, same thickness
Shading:           per-particle vertical gradient
                   top   = base color lightened 18%
                   bottom= base color darkened 12%
Alpha:             0.95–1.0
Drop shadow:       none  (particles are unshadowed in the reference)
```

Provide 3 sprites (capsule, S-curve, J-hook) at 128 px and tint them per color in the particle system.

## 11.4 Motion (Unity ParticleSystem)

```text
Emit:          burst of ~70 at t=0 from a wide band above the screen (x: -10%..110%, y: -5%)
               + low rate ~14/s for 1.8 s afterward; total on-screen ≈ 60
Start speed:   downward 350–900 px/s, plus lateral ±180 px/s
Gravity:       ~+600 px/s²
Drag:          0.5–1.2  (gives the floaty terminal-velocity feel)
Rotation:      random 0–360°, angular velocity ±220°/s
Flutter:       scale X oscillates 1 ↔ 0.35 at 3–6 Hz, random phase (simulates flipping)
Lifetime:      3.0–4.5 s, fade out over the last 0.5 s
Sorting:       above background/heading, below the two buttons
Density cap:   keep the board readable; ≤ 15% of the board area covered at any moment
```

Do not loop confetti forever. One burst plus a short tail is enough.

## 11.5 Monthly Pass note

Per the old spec §3.1, Monthly Pass cosmetics may change confetti only. Keep the shapes and shading; swap only the palette.

---

# 12. Settings screen

**Measured:**

| Element | Value |
|---|---|
| Nav bar | `#FFFFFF`, ends at ≈ y 229 |
| Nav bar bottom | 1-3 px hairline, `#DEDFE1` → `#E3E4E8` |
| Page background | `#EAEDF2` (flat) |
| Card fill | `#FFFFFF` |
| Card top edge | soft: `#ECEDF1` → `#F8F8FA` → `#FFFFFF` over ~3 px |
| Card outer edge | very faint, ≈ `#EEEFF3` 2–3 px halo |
| Row divider (Sound/Vibration) | `#A5AEBF` core, 3–4 px, inset from card edges |
| Toggle track (on) | `#1A79EF` (slight variation `#1876EA`–`#1F7EF6`) |
| Toggle thumb | `#FFFFFF` |
| Chevron | `#A8B0BD` |
| `Done` | `#1869DC` |
| Title | `#152437` |

## 12.1 Polish to add

```text
Nav bar hairline:     keep 1 px #DEDFE1 (already measured) + a 6 px shadow below,
                      rgba(30,45,65,0.06), so the bar lifts off the page
Card shadow:          rgba(30,45,65,0.06), offset (0, -2), blur 8   (very soft; cards are almost flush)
Card radius:          ~40 px on every card (measured: the corner curve reaches the straight edge ~40 px in)
Row divider:          inset ~45 px each side (measured x 86–842), 2 px, #C9CFDA at 80% (softer than measured 3-4 px to avoid heaviness)
Toggle:               track gradient top #2A86F5 → bottom #1A72E8; thumb gets a 1 px shadow rgba(0,0,0,0.20) blur 4
Toggle animation:     thumb slides 0.18 s ease-in-out; track color lerps in the same 0.18 s
Row pressed state:    body flashes #F4F6FA for 0.10 s
Chevron:              rounded caps, 4 px stroke, #A8B0BD
Done:                 pressed → alpha 0.6
```

Card radius is substantially rounder than the old spec's 18–22 px. The measured corner curve reaches the straight edge about 40 px in from the card edge at 945 px wide. Use **~40 px**.

**Measured card and toggle geometry** (945 px reference):

```text
Grouped card (Sound + Vibration):  x 41–887 (846 wide), y 257–519 (263 tall)
Single-row card (Help):            846 wide, 157 tall
Row divider:                       x 86–842  (≈ 45 px inset from each card edge)
Toggle track:                      88 × 56 px, right edge at x 851
Toggle thumb:                      ≈ 44 px (estimate; inset ~6 px inside the track)
```

---

# 13. Micro-interactions and animation polish

All are implementation choices; none are recoverable from stills. They exist to make the build *feel* finished. Keep them restrained, consistent with the old spec's §14.

| Interaction | Effect | Timing |
|---|---|---|
| Button press | scale 0.96, shadow shrinks | 0.08 s |
| Button release | overshoot 1.02 → 1.0 | 0.12 s |
| Piece pick-up (drag start) | scale 1.06, soft shadow appears under piece: `rgba(20,30,50,0.30)`, offset (0,-6), blur 10 | 0.10 s |
| Piece snap into slot | scale 1.06 → 1.0 with 0.98 undershoot | 0.15 s |
| Piece return to bank | ease-out move | 0.18 s |
| Cell fill highlight change | color lerp | ≤ 0.10 s |
| Bank number disable | alpha tween to `#E9E9E9` | 0.15 s |
| Class card select | color lerp + glow fade-in | 0.10 s |
| Hint placed | cell flashes `#FFFFFF` → normal locked blue with a 0.25 s soft ring pulse (`rgba(31,130,252,0.35)`, expands from 1.0× → 1.35×, fades) | 0.25 s |
| Screen transition | cross-fade | 0.20–0.30 s |
| Win overlay | fade in bg, then heading pop, then board, then buttons stagger (0.08 s apart) | 0.3 s total |
| Hint badge change | numeral scale pop 1.0 → 1.25 → 1.0 | 0.18 s |

**Do not** add:

- continuous pulsing on the orange class focus;
- screen shake;
- bounce on grid cells.

---

# 14. Shadow and glow reference sheet

One place to look up every elevation value.

| Element | Type | Color | Offset | Blur |
|---|---|---|---|---|
| Bank card | drop | `rgba(30,45,65,0.30)` | (0, -6) | 14 |
| Bank card, contact | drop | `rgba(20,30,50,0.35)` | (0, -3) | 4 |
| Selected class card | glow | `rgba(255,200,80,0.45)` | (0, 0) | 16 |
| Toolbar circle | drop | `rgba(49,102,175,0.18)` | (0, -4) | 14 |
| Blue primary button | colored drop | `rgba(31,130,252,0.35)` | (0, -8) | 20 |
| White win button | outer glow | `rgba(120,170,255,0.35)` | (0, -4) | 18 |
| Logo | drop | `rgba(52,68,92,0.18)` | (0, -3) | 8 |
| Settings card | drop | `rgba(30,45,65,0.06)` | (0, -2) | 8 |
| Settings nav bar | drop | `rgba(30,45,65,0.06)` | (0, -1) | 6 |
| Dragged piece | drop | `rgba(20,30,50,0.30)` | (0, -6) | 10 |
| Congratulations | underlay | `rgba(20,40,110,0.45)` | (0, -5) | 6 |
| Board | **none** | | | |
| Grid cells | **none** | | | |

Sign convention: negative Y means "downward on screen" (per the old spec's note). If Unity's `Shadow` component renders the opposite, invert Y.

---

# 15. Gradient reference sheet

| Surface | Type | Stops |
|---|---|---|
| Gameplay bg | vertical | `#EBEFFA` → `#D8E2EE` |
| Main bg | vertical | `#EFF6FE` → `#DCECFC` (y 1800) → `#D4EAFF` |
| Win bg | radial | `#1877F5` (0%) → `#1A62D1` (35%) → `#1B4FB5` (70%) → `#1C44A4` (100%) |
| Congratulations fill | vertical | `#FCE283` → `#FAD16C` → `#F8BD4C` |
| Blue button | vertical | `#2D8CFF` → `#1A78F0` |
| Toggle track | vertical | `#2A86F5` → `#1A72E8` |
| Confetti | per-particle vertical | base +18% lightness → base −12% |
| Main puzzle decoration | mask (vertical) | alpha 1.0 (y=0) → 0 (y≈1250) |

---

# 16. Updated color tokens

Additions and changes to the token table in `VISUAL_DESIGN_SPEC.md` §3.

| Token | Hex | Status | Use |
|---|---|---|---|
| `GameplayBackgroundTop` | `#EBEFFA` | measured | top of gameplay gradient (new) |
| `GameplayBackground` | `#D8E2EE` | measured | bottom stop (was `#DAE4F0`) |
| `MainBackgroundTop` | `#EFF6FE` | measured | top of main gradient (new) |
| `MainBackgroundBottom` | `#D4EAFF` | measured | bottom of main gradient (new) |
| `WinBgCenter` | `#1877F5` | measured | radial center (new) |
| `WinBgMid` | `#1A62D1` | measured | radial mid (new) |
| `WinBgEdge` | `#1D49AC` | measured | radial edge, top (new) |
| `WinBgBottom` | `#1C44A4` | measured | bottom (new) |
| `HeadingFillTop` | `#FCE283` | measured | Congratulations top (new) |
| `HeadingFillBottom` | `#F8BD4C` | measured | Congratulations bottom (new) |
| `HeadingOutline` | `#88432A` | measured | Congratulations outline (new) |
| `BoardFrameOuter` | `#152837` | measured | board outer band (new) |
| `GridStroke` | `#0A0F14` | measured (~`#000`–`#101010`) | cell lines (was `#101820`) |
| `ToolbarCircle` | `#D2DFF0` | measured | (was `#D0DEF0`, effectively the same) |
| `ToolbarIconBlue` | `#24468E` | measured | icon strokes (was `#3166AF`, lighter) |
| `SettingsCardEdge` | `#EEEFF3` | measured | faint halo on settings cards (new) |
| `SettingsToggleOn` | `#1A79EF` | measured | toggle track (was `#1F82FC`) |
| `SettingsDone` | `#1869DC` | measured | Done label (new) |
| `SettingsTitle` | `#152437` | measured | Settings title (new) |
| `SettingsChevron` | `#A8B0BD` | measured | chevrons (new) |
| `WinButtonLabel` | `#2B4568` | measured | white-pill label (new) |
| `MainDecoBlocked` | `#C9D8EE` | implementation choice | blocked cells in main decoration |
| `MainDecoDigit` | `#C7D0DD` | implementation choice | faded digits in main decoration |
| `ConfettiRed` | `#F0284F` | observed | confetti |
| `ConfettiMagenta` | `#F02ED0` | observed | confetti |
| `ConfettiGreen` | `#22D05A` | observed | confetti |
| `ConfettiYellow` | `#FFEA1F` | observed | confetti |
| `ConfettiOrange` | `#FF8A1F` | observed | confetti |
| `ConfettiCyan` | `#3EE8F2` | observed | confetti |
| `ConfettiPurple` | `#7A3DF0` | observed | confetti |
| `ConfettiBlue` | `#2A8BFF` | observed | confetti |

The semantic colors (locked blue, orange, green, red, blocked navy) are **unchanged**. See §17 of this file for the guardrail.

---

# 17. Semantic-color guardrail (unchanged)

All added polish is for chrome only. The following must remain exactly as defined in `VISUAL_DESIGN_SPEC.md` §3.1 and must never receive a gradient, glow, outline, or shadow:

```text
Locked blue   #C9DAEC   (cell fill)
White         #FFFFFF   (cell fill)
Orange        #FFE6A3   (slot fill), #FFDB83/#FFF8E5 (class card only)
Green         #BCE9C0 / #9FE7A5
Red/pink      #FFD1D4
Blocked navy  #28394D
```

The class-card glow in §6.4 applies to the **card in the bank**, never to the board slots.

---

# 18. Unity implementation notes

## 18.1 Suggested components

| Need | Unity approach |
|---|---|
| Vertical gradient background | UI `Image` with a 1 × 256 baked gradient sprite, or a shader-less `Gradient` component |
| Radial gradient | baked 256 × 512 sprite, `Image` set to Simple, stretched to fill |
| Rounded pill/card | 9-sliced sprite (`Sprite Editor` borders), so radius survives resizing |
| Soft shadows | pre-blurred 9-sliced shadow sprite behind the element (cheaper and smoother than `UnityEngine.UI.Shadow`, and works on WebGL) |
| Hero text | TextMeshPro SDF: Color Gradient + Outline + Underlay (§10.4) |
| Confetti | `ParticleSystem` with a 3-frame sprite sheet (capsule, S, J) + tint via `Start Color` random between gradient |
| Main decoration | baked sprite; skip live perspective transforms |
| Button press | a small `PressableScale` MonoBehaviour driven by `IPointerDown/Up` |

## 18.2 WebGL performance

- Bake every static gradient and shadow to a sprite; do **not** use per-frame UI effects on WebGL.
- Confetti: max ~90 live particles, one material, one texture atlas.
- Avoid full-screen blur/post-processing.
- Keep the main decoration to one draw call.

## 18.3 Config additions

Extend `GameplayVisualConfig` (old §16) with:

```text
Color gameplayBackgroundTop
Color mainBackgroundTop, mainBackgroundBottom
Color winBgCenter, winBgMid, winBgEdge, winBgBottom
Color headingFillTop, headingFillBottom, headingOutline
Color boardFrameOuter
Color[] confettiPalette (8 entries, §11.2)

float bankHeaderHeight = 96
float pillButtonHeight = 144
float pillButtonWidthRatio = 0.69     // 658 / 945
float toolbarCircleDiameter = 82
float cardCornerRadius = 14
float settingsCardRadius = 40
```

Everything else can stay on individual views.

---

# 19. Screenshot-driven acceptance checklist

Compare against the reference screenshots side by side before marking a screen done.

## Gameplay

- [ ] background is a soft top-to-bottom gradient, not flat
- [ ] Poppins used for bank headers and numbers
- [ ] bank cards visibly lift off the background (shadow darkest directly under the card)
- [ ] bank header ≈ 96 px tall at reference
- [ ] selected class card has orange border + soft glow
- [ ] board has navy outer band + black inner line, no shadow
- [ ] cells are still perfectly flat
- [ ] toolbar circles are `#D2DFF0` with a very soft shadow
- [ ] hint badge has a white ring

## Main

- [ ] vertical gradient ending in saturated pale blue at the bottom
- [ ] decoration is a tilted, perspective puzzle with blocked cells **and digits**
- [ ] decoration fades out before the button
- [ ] logo shows the `✕` in the circle, in `#34445C`
- [ ] gear icon clearly legible over the decoration
- [ ] New Game pill ≈ 658 × 144, gradient fill, blue-tinted shadow
- [ ] pressing the button scales it down and back with a tiny overshoot

## Settings

- [ ] very round cards (≈ 40 px radius), white on `#EAEDF2`
- [ ] nav bar has hairline + faint shadow
- [ ] toggles are `#1A79EF` with a white thumb and a small thumb shadow
- [ ] `Done` is `#1869DC`
- [ ] chevrons are light cool gray

## Win

- [ ] background is a **radial** gradient, brightest just above center
- [ ] `Congratulations!` has vertical gold gradient **and** brown outline **and** drop shadow
- [ ] confetti has all 7 colors and 3 shapes with inner shading
- [ ] confetti overlaps the heading/board edges slightly, sits under the buttons
- [ ] white pill has a soft blue glow, label `#2B4568`
- [ ] `Main` is white Poppins SemiBold with a soft text shadow
- [ ] confetti clears the board; the completed board is fully readable

---

# 20. Authority rule

1. Screenshot-observed **semantics** in `GAME_SPEC.md` still win.
2. Where this file and `VISUAL_DESIGN_SPEC.md` disagree, **this file wins** for the items listed in §1.1 and in the tables of §16.
3. Anything not mentioned here falls back to `VISUAL_DESIGN_SPEC.md`.
4. `monthly_pass.md` may recolor confetti and add Premium gold accents, but cannot override gameplay semantic colors or remove the chrome polish defined here.
5. When a value is marked *implementation choice*, prefer matching the screenshots over matching the number.
