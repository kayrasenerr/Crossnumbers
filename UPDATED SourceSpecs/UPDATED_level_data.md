# Crossnumbers — Level Data

**Target:** Unity 2022 LTS+  
**Purpose:** Single source of level-authoring data for all six case-study levels.  
**Companion specs:** `GAME_SPEC.md`, `IMPLEMENTATION_SPEC.md`, `monthly_pass.md`, `VISUAL_DESIGN_SPEC.md`

This file contains the concrete data needed to create the six `LevelDefinition` ScriptableObjects. It does not redefine gameplay rules.

---

# 1. Coordinate and data conventions

All explicit coordinates and slot digit indices in this file are **zero-based**, matching the recommended Unity runtime representation.

- `row: 0` = first/top row.
- `column: 0` = first/left column.
- Slot `digitIndex: 0` = first digit of the number in reading direction.
- Horizontal slots read left → right.
- Vertical slots read top → bottom.

Grid strings use:

- `#` = blocked cell.
- `.` = empty active cell.
- `0`–`9` = game-supplied locked light-blue digit.

For each level, build the `LevelDefinition` fields as follows:

```text
rows / columns       <- metadata below
blockedCells         <- every # position in initialGrid
initialLockedCells   <- every digit position in initialGrid
slots                <- exact slot records below
pieces               <- exact piece records below
solutionCells        <- every digit position in solutionGrid
```

The number bank UI should group `pieces` by `digits.Length`; grouping does not need separate authored data.

`canonicalPieceId` is included for verification, editor tooling, tests, and creation of `solutionGrid`. Runtime win detection should still use the gameplay rules in `GAME_SPEC.md`, not require the canonical slot assignment unless the implementation deliberately chooses that simpler equivalent.

---

# 2. Validation summary

| Level | Grid | Pieces / slots | Intersections | Initial locked digits | Active cells |
|---|---:|---:|---:|---:|---:|
| 1 | 5×5 | 2 | 1 | 2 | 9 |
| 2 | 6×6 | 6 | 6 | 12 | 24 |
| 3 | 6×6 | 6 | 6 | 12 | 24 |
| 4 | 8×8 | 10 | 10 | 13 | 40 |
| 5 | 8×8 | 10 | 10 | 14 | 40 |
| 6 | 9×9 | 15 | 22 | 11 | 55 |

All six datasets were cross-checked for these invariants:

1. Every piece length equals its canonical slot length.
2. Every canonical piece appears exactly once.
3. Every slot receives exactly one canonical piece.
4. All canonical crossing digits agree.
5. Every non-`#` initial-grid cell belongs to at least one slot.
6. No `#` cell belongs to a slot.
7. Every initial locked digit agrees with the canonical solution.
8. Every intersection below is derived from the authored slot geometry.

---

# 3. Recommended C# authoring shapes

These names match `IMPLEMENTATION_SPEC.md`.

```csharp
[Serializable]
public struct CellCoord
{
    public int row;
    public int column;
}

[Serializable]
public struct LockedCellDefinition
{
    public CellCoord coord;
    public char digit;
}

public enum SlotOrientation
{
    Horizontal,
    Vertical
}

[Serializable]
public class SlotDefinition
{
    public string slotId;
    public SlotOrientation orientation;
    public CellCoord start;
    public int length;
}

[Serializable]
public class PieceDefinition
{
    public string pieceId;
    public string digits;
}

[Serializable]
public struct SolutionCellDefinition
{
    public CellCoord coord;
    public char digit;
}
```

`IntersectionDefinition` does not need to be serialized if intersections are derived at load time. If useful for tests/debugging, use:

```csharp
[Serializable]
public struct IntersectionDefinition
{
    public CellCoord coord;
    public string slotA;
    public int slotADigitIndex;
    public string slotB;
    public int slotBDigitIndex;
}
```

---

# 4. Level 1 — `Level_01`

**Status:** Source-derived tutorial reconstruction.

## Metadata

```yaml
levelId: "Level_01"
rows: 5
columns: 5
isTutorial: true
showToolbar: false
pieceCount: 2
slotCount: 2
intersectionCount: 1
initialLockedCount: 2
```

## Initial grid

```text
##5##
##.##
1....
##.##
##.##
```

### Initial locked cells

| row | column | digit |
|---:|---:|:---:|
| 0 | 2 | `5` |
| 2 | 0 | `1` |

## Pieces

| pieceId | digits | length |
|---|---:|---:|
| `Level_01_P01` | `12345` | 5 |
| `Level_01_P02` | `54321` | 5 |

## Slots

| slotId | orientation | start `(row,col)` | length | canonicalPieceId | canonical digits |
|---|---|---:|---:|---|---:|
| `H1` | Horizontal | `(2,0)` | 5 | `Level_01_P01` | `12345` |
| `V1` | Vertical | `(0,2)` | 5 | `Level_01_P02` | `54321` |

### Expanded slot cells

Useful for debugging/import verification; these may be derived at runtime rather than serialized.

- `H1`: (2,0), (2,1), (2,2), (2,3), (2,4)
- `V1`: (0,2), (1,2), (2,2), (3,2), (4,2)

## Intersections

Each index is the zero-based character index inside that slot's piece.

| cell `(row,col)` | slot A | A digitIndex | slot B | B digitIndex | canonical digit |
|---:|---|---:|---|---:|:---:|
| `(2,2)` | `H1` | 2 | `V1` | 2 | `3` |

## Canonical solution grid

`#` cells remain blocked. Every other character becomes a `SolutionCellDefinition` entry for Hint/win verification.

```text
##5##
##4##
12345
##2##
##1##
```

## Canonical assignment map

```text
H1 -> Level_01_P01 -> 12345
V1 -> Level_01_P02 -> 54321
```

---

# 5. Level 2 — `Level_02`

**Status:** Source-derived/reconstructed solved level.

## Metadata

```yaml
levelId: "Level_02"
rows: 6
columns: 6
isTutorial: false
showToolbar: true
pieceCount: 6
slotCount: 6
intersectionCount: 6
initialLockedCount: 12
```

## Initial grid

```text
.3..4.
#5##2#
#7#.9.
.0.#1#
#4##5#
.0..6.
```

### Initial locked cells

| row | column | digit |
|---:|---:|:---:|
| 0 | 1 | `3` |
| 0 | 4 | `4` |
| 1 | 1 | `5` |
| 1 | 4 | `2` |
| 2 | 1 | `7` |
| 2 | 4 | `9` |
| 3 | 1 | `0` |
| 3 | 4 | `1` |
| 4 | 1 | `4` |
| 4 | 4 | `5` |
| 5 | 1 | `0` |
| 5 | 4 | `6` |

## Pieces

| pieceId | digits | length |
|---|---:|---:|
| `Level_02_P01` | `301` | 3 |
| `Level_02_P02` | `792` | 3 |
| `Level_02_P03` | `309165` | 6 |
| `Level_02_P04` | `357040` | 6 |
| `Level_02_P05` | `429156` | 6 |
| `Level_02_P06` | `734748` | 6 |

## Slots

| slotId | orientation | start `(row,col)` | length | canonicalPieceId | canonical digits |
|---|---|---:|---:|---|---:|
| `H1` | Horizontal | `(0,0)` | 6 | `Level_02_P06` | `734748` |
| `H2` | Horizontal | `(2,3)` | 3 | `Level_02_P02` | `792` |
| `H3` | Horizontal | `(3,0)` | 3 | `Level_02_P01` | `301` |
| `H4` | Horizontal | `(5,0)` | 6 | `Level_02_P03` | `309165` |
| `V1` | Vertical | `(0,1)` | 6 | `Level_02_P04` | `357040` |
| `V2` | Vertical | `(0,4)` | 6 | `Level_02_P05` | `429156` |

### Expanded slot cells

Useful for debugging/import verification; these may be derived at runtime rather than serialized.

- `H1`: (0,0), (0,1), (0,2), (0,3), (0,4), (0,5)
- `H2`: (2,3), (2,4), (2,5)
- `H3`: (3,0), (3,1), (3,2)
- `H4`: (5,0), (5,1), (5,2), (5,3), (5,4), (5,5)
- `V1`: (0,1), (1,1), (2,1), (3,1), (4,1), (5,1)
- `V2`: (0,4), (1,4), (2,4), (3,4), (4,4), (5,4)

## Intersections

Each index is the zero-based character index inside that slot's piece.

| cell `(row,col)` | slot A | A digitIndex | slot B | B digitIndex | canonical digit |
|---:|---|---:|---|---:|:---:|
| `(0,1)` | `H1` | 1 | `V1` | 0 | `3` |
| `(0,4)` | `H1` | 4 | `V2` | 0 | `4` |
| `(2,4)` | `H2` | 1 | `V2` | 2 | `9` |
| `(3,1)` | `H3` | 1 | `V1` | 3 | `0` |
| `(5,1)` | `H4` | 1 | `V1` | 5 | `0` |
| `(5,4)` | `H4` | 4 | `V2` | 5 | `6` |

## Canonical solution grid

`#` cells remain blocked. Every other character becomes a `SolutionCellDefinition` entry for Hint/win verification.

```text
734748
#5##2#
#7#792
301#1#
#4##5#
309165
```

## Canonical assignment map

```text
H1 -> Level_02_P06 -> 734748
H2 -> Level_02_P02 -> 792
H3 -> Level_02_P01 -> 301
H4 -> Level_02_P03 -> 309165
V1 -> Level_02_P04 -> 357040
V2 -> Level_02_P05 -> 429156
```

---

# 6. Level 3 — `Level_03`

**Status:** Screenshot-verified: initial and solved states transcribed directly from level screenshots.

## Metadata

```yaml
levelId: "Level_03"
rows: 6
columns: 6
isTutorial: false
showToolbar: true
pieceCount: 6
slotCount: 6
intersectionCount: 6
initialLockedCount: 12
```

## Initial grid

```text
.##.#.
103590
.##.#.
.#.##.
791864
.#.##.
```

### Initial locked cells

| row | column | digit |
|---:|---:|:---:|
| 1 | 0 | `1` |
| 1 | 1 | `0` |
| 1 | 2 | `3` |
| 1 | 3 | `5` |
| 1 | 4 | `9` |
| 1 | 5 | `0` |
| 4 | 0 | `7` |
| 4 | 1 | `9` |
| 4 | 2 | `1` |
| 4 | 3 | `8` |
| 4 | 4 | `6` |
| 4 | 5 | `4` |

## Pieces

| pieceId | digits | length |
|---|---:|---:|
| `Level_03_P01` | `656` | 3 |
| `Level_03_P02` | `913` | 3 |
| `Level_03_P03` | `103590` | 6 |
| `Level_03_P04` | `207340` | 6 |
| `Level_03_P05` | `711479` | 6 |
| `Level_03_P06` | `791864` | 6 |

## Slots

| slotId | orientation | start `(row,col)` | length | canonicalPieceId | canonical digits |
|---|---|---:|---:|---|---:|
| `H1` | Horizontal | `(1,0)` | 6 | `Level_03_P03` | `103590` |
| `H2` | Horizontal | `(4,0)` | 6 | `Level_03_P06` | `791864` |
| `V1` | Vertical | `(0,0)` | 6 | `Level_03_P05` | `711479` |
| `V2` | Vertical | `(3,2)` | 3 | `Level_03_P02` | `913` |
| `V3` | Vertical | `(0,3)` | 3 | `Level_03_P01` | `656` |
| `V4` | Vertical | `(0,5)` | 6 | `Level_03_P04` | `207340` |

### Expanded slot cells

Useful for debugging/import verification; these may be derived at runtime rather than serialized.

- `H1`: (1,0), (1,1), (1,2), (1,3), (1,4), (1,5)
- `H2`: (4,0), (4,1), (4,2), (4,3), (4,4), (4,5)
- `V1`: (0,0), (1,0), (2,0), (3,0), (4,0), (5,0)
- `V2`: (3,2), (4,2), (5,2)
- `V3`: (0,3), (1,3), (2,3)
- `V4`: (0,5), (1,5), (2,5), (3,5), (4,5), (5,5)

## Intersections

Each index is the zero-based character index inside that slot's piece.

| cell `(row,col)` | slot A | A digitIndex | slot B | B digitIndex | canonical digit |
|---:|---|---:|---|---:|:---:|
| `(1,0)` | `H1` | 0 | `V1` | 1 | `1` |
| `(1,3)` | `H1` | 3 | `V3` | 1 | `5` |
| `(1,5)` | `H1` | 5 | `V4` | 1 | `0` |
| `(4,0)` | `H2` | 0 | `V1` | 4 | `7` |
| `(4,2)` | `H2` | 2 | `V2` | 1 | `1` |
| `(4,5)` | `H2` | 5 | `V4` | 4 | `4` |

## Canonical solution grid

`#` cells remain blocked. Every other character becomes a `SolutionCellDefinition` entry for Hint/win verification.

```text
7##6#2
103590
1##6#7
4#9##3
791864
9#3##0
```

## Canonical assignment map

```text
H1 -> Level_03_P03 -> 103590
H2 -> Level_03_P06 -> 791864
V1 -> Level_03_P05 -> 711479
V2 -> Level_03_P02 -> 913
V3 -> Level_03_P01 -> 656
V4 -> Level_03_P04 -> 207340
```

---

# 7. Level 4 — `Level_04`

**Status:** Screenshot-verified: initial and solved states transcribed directly from level screenshots.

## Metadata

```yaml
levelId: "Level_04"
rows: 8
columns: 8
isTutorial: false
showToolbar: true
pieceCount: 10
slotCount: 10
intersectionCount: 10
initialLockedCount: 13
```

## Initial grid

```text
##....2.
#2#.##1#
.....#0#
#2#.#.3.
.9.#.#3#
#3#...0.
#9##.#2#
.0....##
```

### Initial locked cells

| row | column | digit |
|---:|---:|:---:|
| 0 | 6 | `2` |
| 1 | 1 | `2` |
| 1 | 6 | `1` |
| 2 | 6 | `0` |
| 3 | 1 | `2` |
| 3 | 6 | `3` |
| 4 | 1 | `9` |
| 4 | 6 | `3` |
| 5 | 1 | `3` |
| 5 | 6 | `0` |
| 6 | 1 | `9` |
| 6 | 6 | `2` |
| 7 | 1 | `0` |

## Pieces

| pieceId | digits | length |
|---|---:|---:|
| `Level_04_P01` | `239` | 3 |
| `Level_04_P02` | `892` | 3 |
| `Level_04_P03` | `1002` | 4 |
| `Level_04_P04` | `6643` | 4 |
| `Level_04_P05` | `13949` | 5 |
| `Level_04_P06` | `90409` | 5 |
| `Level_04_P07` | `105522` | 6 |
| `Level_04_P08` | `867122` | 6 |
| `Level_04_P09` | `2103302` | 7 |
| `Level_04_P10` | `2329390` | 7 |

## Slots

| slotId | orientation | start `(row,col)` | length | canonicalPieceId | canonical digits |
|---|---|---:|---:|---|---:|
| `H1` | Horizontal | `(0,2)` | 6 | `Level_04_P08` | `867122` |
| `H2` | Horizontal | `(2,0)` | 5 | `Level_04_P05` | `13949` |
| `H3` | Horizontal | `(3,5)` | 3 | `Level_04_P01` | `239` |
| `H4` | Horizontal | `(4,0)` | 3 | `Level_04_P02` | `892` |
| `H5` | Horizontal | `(5,3)` | 5 | `Level_04_P06` | `90409` |
| `H6` | Horizontal | `(7,0)` | 6 | `Level_04_P07` | `105522` |
| `V1` | Vertical | `(1,1)` | 7 | `Level_04_P10` | `2329390` |
| `V2` | Vertical | `(0,3)` | 4 | `Level_04_P04` | `6643` |
| `V3` | Vertical | `(4,4)` | 4 | `Level_04_P03` | `1002` |
| `V4` | Vertical | `(0,6)` | 7 | `Level_04_P09` | `2103302` |

### Expanded slot cells

Useful for debugging/import verification; these may be derived at runtime rather than serialized.

- `H1`: (0,2), (0,3), (0,4), (0,5), (0,6), (0,7)
- `H2`: (2,0), (2,1), (2,2), (2,3), (2,4)
- `H3`: (3,5), (3,6), (3,7)
- `H4`: (4,0), (4,1), (4,2)
- `H5`: (5,3), (5,4), (5,5), (5,6), (5,7)
- `H6`: (7,0), (7,1), (7,2), (7,3), (7,4), (7,5)
- `V1`: (1,1), (2,1), (3,1), (4,1), (5,1), (6,1), (7,1)
- `V2`: (0,3), (1,3), (2,3), (3,3)
- `V3`: (4,4), (5,4), (6,4), (7,4)
- `V4`: (0,6), (1,6), (2,6), (3,6), (4,6), (5,6), (6,6)

## Intersections

Each index is the zero-based character index inside that slot's piece.

| cell `(row,col)` | slot A | A digitIndex | slot B | B digitIndex | canonical digit |
|---:|---|---:|---|---:|:---:|
| `(0,3)` | `H1` | 1 | `V2` | 0 | `6` |
| `(0,6)` | `H1` | 4 | `V4` | 0 | `2` |
| `(2,1)` | `H2` | 1 | `V1` | 1 | `3` |
| `(2,3)` | `H2` | 3 | `V2` | 2 | `4` |
| `(3,6)` | `H3` | 1 | `V4` | 3 | `3` |
| `(4,1)` | `H4` | 1 | `V1` | 3 | `9` |
| `(5,4)` | `H5` | 1 | `V3` | 1 | `0` |
| `(5,6)` | `H5` | 3 | `V4` | 5 | `0` |
| `(7,1)` | `H6` | 1 | `V1` | 6 | `0` |
| `(7,4)` | `H6` | 4 | `V3` | 3 | `2` |

## Canonical solution grid

`#` cells remain blocked. Every other character becomes a `SolutionCellDefinition` entry for Hint/win verification.

```text
##867122
#2#6##1#
13949#0#
#2#3#239
892#1#3#
#3#90409
#9##0#2#
105522##
```

## Canonical assignment map

```text
H1 -> Level_04_P08 -> 867122
H2 -> Level_04_P05 -> 13949
H3 -> Level_04_P01 -> 239
H4 -> Level_04_P02 -> 892
H5 -> Level_04_P06 -> 90409
H6 -> Level_04_P07 -> 105522
V1 -> Level_04_P10 -> 2329390
V2 -> Level_04_P04 -> 6643
V3 -> Level_04_P03 -> 1002
V4 -> Level_04_P09 -> 2103302
```

---

# 8. Level 5 — `Level_05`

**Status:** Screenshot-verified: initial and solved states transcribed directly from level screenshots.

## Metadata

```yaml
levelId: "Level_05"
rows: 8
columns: 8
isTutorial: false
showToolbar: true
pieceCount: 10
slotCount: 10
intersectionCount: 10
initialLockedCount: 14
```

## Initial grid

```text
##.#.##.
#9454436
.#.#.##.
....#.#.
.#.#....
.##.#.#.
7200230#
.##.#.##
```

### Initial locked cells

| row | column | digit |
|---:|---:|:---:|
| 1 | 1 | `9` |
| 1 | 2 | `4` |
| 1 | 3 | `5` |
| 1 | 4 | `4` |
| 1 | 5 | `4` |
| 1 | 6 | `3` |
| 1 | 7 | `6` |
| 6 | 0 | `7` |
| 6 | 1 | `2` |
| 6 | 2 | `0` |
| 6 | 3 | `0` |
| 6 | 4 | `2` |
| 6 | 5 | `3` |
| 6 | 6 | `0` |

## Pieces

| pieceId | digits | length |
|---|---:|---:|
| `Level_05_P01` | `409` | 3 |
| `Level_05_P02` | `445` | 3 |
| `Level_05_P03` | `2305` | 4 |
| `Level_05_P04` | `7567` | 4 |
| `Level_05_P05` | `43232` | 5 |
| `Level_05_P06` | `64269` | 5 |
| `Level_05_P07` | `163753` | 6 |
| `Level_05_P08` | `670473` | 6 |
| `Level_05_P09` | `7200230` | 7 |
| `Level_05_P10` | `9454436` | 7 |

## Slots

| slotId | orientation | start `(row,col)` | length | canonicalPieceId | canonical digits |
|---|---|---:|---:|---|---:|
| `H1` | Horizontal | `(1,1)` | 7 | `Level_05_P10` | `9454436` |
| `H2` | Horizontal | `(3,0)` | 4 | `Level_05_P04` | `7567` |
| `H3` | Horizontal | `(4,4)` | 4 | `Level_05_P03` | `2305` |
| `H4` | Horizontal | `(6,0)` | 7 | `Level_05_P09` | `7200230` |
| `V1` | Vertical | `(2,0)` | 6 | `Level_05_P08` | `670473` |
| `V2` | Vertical | `(0,2)` | 5 | `Level_05_P06` | `64269` |
| `V3` | Vertical | `(5,3)` | 3 | `Level_05_P01` | `409` |
| `V4` | Vertical | `(0,4)` | 3 | `Level_05_P02` | `445` |
| `V5` | Vertical | `(3,5)` | 5 | `Level_05_P05` | `43232` |
| `V6` | Vertical | `(0,7)` | 6 | `Level_05_P07` | `163753` |

### Expanded slot cells

Useful for debugging/import verification; these may be derived at runtime rather than serialized.

- `H1`: (1,1), (1,2), (1,3), (1,4), (1,5), (1,6), (1,7)
- `H2`: (3,0), (3,1), (3,2), (3,3)
- `H3`: (4,4), (4,5), (4,6), (4,7)
- `H4`: (6,0), (6,1), (6,2), (6,3), (6,4), (6,5), (6,6)
- `V1`: (2,0), (3,0), (4,0), (5,0), (6,0), (7,0)
- `V2`: (0,2), (1,2), (2,2), (3,2), (4,2)
- `V3`: (5,3), (6,3), (7,3)
- `V4`: (0,4), (1,4), (2,4)
- `V5`: (3,5), (4,5), (5,5), (6,5), (7,5)
- `V6`: (0,7), (1,7), (2,7), (3,7), (4,7), (5,7)

## Intersections

Each index is the zero-based character index inside that slot's piece.

| cell `(row,col)` | slot A | A digitIndex | slot B | B digitIndex | canonical digit |
|---:|---|---:|---|---:|:---:|
| `(1,2)` | `H1` | 1 | `V2` | 1 | `4` |
| `(1,4)` | `H1` | 3 | `V4` | 1 | `4` |
| `(1,7)` | `H1` | 6 | `V6` | 1 | `6` |
| `(3,0)` | `H2` | 0 | `V1` | 1 | `7` |
| `(3,2)` | `H2` | 2 | `V2` | 3 | `6` |
| `(4,5)` | `H3` | 1 | `V5` | 1 | `3` |
| `(4,7)` | `H3` | 3 | `V6` | 4 | `5` |
| `(6,0)` | `H4` | 0 | `V1` | 4 | `7` |
| `(6,3)` | `H4` | 3 | `V3` | 1 | `0` |
| `(6,5)` | `H4` | 5 | `V5` | 3 | `3` |

## Canonical solution grid

`#` cells remain blocked. Every other character becomes a `SolutionCellDefinition` entry for Hint/win verification.

```text
##6#4##1
#9454436
6#2#5##3
7567#4#7
0#9#2305
4##4#2#3
7200230#
3##9#2##
```

## Canonical assignment map

```text
H1 -> Level_05_P10 -> 9454436
H2 -> Level_05_P04 -> 7567
H3 -> Level_05_P03 -> 2305
H4 -> Level_05_P09 -> 7200230
V1 -> Level_05_P08 -> 670473
V2 -> Level_05_P06 -> 64269
V3 -> Level_05_P01 -> 409
V4 -> Level_05_P02 -> 445
V5 -> Level_05_P05 -> 43232
V6 -> Level_05_P07 -> 163753
```

---

# 9. Level 6 — `Level_06`

**Status:** Screenshot-verified: initial and solved states transcribed directly from level screenshots.

## Metadata

```yaml
levelId: "Level_06"
rows: 9
columns: 9
isTutorial: false
showToolbar: true
pieceCount: 15
slotCount: 15
intersectionCount: 22
initialLockedCount: 11
```

## Initial grid

```text
....####.
#.##1....
66209#.#.
.#.#3....
.#.#1#.#.
....7#.#.
.#.#2....
....5##.#
.####....
```

### Initial locked cells

| row | column | digit |
|---:|---:|:---:|
| 1 | 4 | `1` |
| 2 | 0 | `6` |
| 2 | 1 | `6` |
| 2 | 2 | `2` |
| 2 | 3 | `0` |
| 2 | 4 | `9` |
| 3 | 4 | `3` |
| 4 | 4 | `1` |
| 5 | 4 | `7` |
| 6 | 4 | `2` |
| 7 | 4 | `5` |

## Pieces

| pieceId | digits | length |
|---|---:|---:|
| `Level_06_P01` | `496` | 3 |
| `Level_06_P02` | `651` | 3 |
| `Level_06_P03` | `6211` | 4 |
| `Level_06_P04` | `8423` | 4 |
| `Level_06_P05` | `17142` | 5 |
| `Level_06_P06` | `25263` | 5 |
| `Level_06_P07` | `39096` | 5 |
| `Level_06_P08` | `66209` | 5 |
| `Level_06_P09` | `71247` | 5 |
| `Level_06_P10` | `79285` | 5 |
| `Level_06_P11` | `120302` | 6 |
| `Level_06_P12` | `266282` | 6 |
| `Level_06_P13` | `1931725` | 7 |
| `Level_06_P14` | `6017778` | 7 |
| `Level_06_P15` | `9256303` | 7 |

## Slots

| slotId | orientation | start `(row,col)` | length | canonicalPieceId | canonical digits |
|---|---|---:|---:|---|---:|
| `H1` | Horizontal | `(0,0)` | 4 | `Level_06_P04` | `8423` |
| `H2` | Horizontal | `(1,4)` | 5 | `Level_06_P05` | `17142` |
| `H3` | Horizontal | `(2,0)` | 5 | `Level_06_P08` | `66209` |
| `H4` | Horizontal | `(3,4)` | 5 | `Level_06_P07` | `39096` |
| `H5` | Horizontal | `(5,0)` | 5 | `Level_06_P09` | `71247` |
| `H6` | Horizontal | `(6,4)` | 5 | `Level_06_P06` | `25263` |
| `H7` | Horizontal | `(7,0)` | 5 | `Level_06_P10` | `79285` |
| `H8` | Horizontal | `(8,5)` | 4 | `Level_06_P03` | `6211` |
| `V1` | Vertical | `(2,0)` | 7 | `Level_06_P14` | `6017778` |
| `V2` | Vertical | `(0,1)` | 3 | `Level_06_P01` | `496` |
| `V3` | Vertical | `(2,2)` | 6 | `Level_06_P12` | `266282` |
| `V4` | Vertical | `(1,4)` | 7 | `Level_06_P13` | `1931725` |
| `V5` | Vertical | `(1,6)` | 6 | `Level_06_P11` | `120302` |
| `V6` | Vertical | `(6,7)` | 3 | `Level_06_P02` | `651` |
| `V7` | Vertical | `(0,8)` | 7 | `Level_06_P15` | `9256303` |

### Expanded slot cells

Useful for debugging/import verification; these may be derived at runtime rather than serialized.

- `H1`: (0,0), (0,1), (0,2), (0,3)
- `H2`: (1,4), (1,5), (1,6), (1,7), (1,8)
- `H3`: (2,0), (2,1), (2,2), (2,3), (2,4)
- `H4`: (3,4), (3,5), (3,6), (3,7), (3,8)
- `H5`: (5,0), (5,1), (5,2), (5,3), (5,4)
- `H6`: (6,4), (6,5), (6,6), (6,7), (6,8)
- `H7`: (7,0), (7,1), (7,2), (7,3), (7,4)
- `H8`: (8,5), (8,6), (8,7), (8,8)
- `V1`: (2,0), (3,0), (4,0), (5,0), (6,0), (7,0), (8,0)
- `V2`: (0,1), (1,1), (2,1)
- `V3`: (2,2), (3,2), (4,2), (5,2), (6,2), (7,2)
- `V4`: (1,4), (2,4), (3,4), (4,4), (5,4), (6,4), (7,4)
- `V5`: (1,6), (2,6), (3,6), (4,6), (5,6), (6,6)
- `V6`: (6,7), (7,7), (8,7)
- `V7`: (0,8), (1,8), (2,8), (3,8), (4,8), (5,8), (6,8)

## Intersections

Each index is the zero-based character index inside that slot's piece.

| cell `(row,col)` | slot A | A digitIndex | slot B | B digitIndex | canonical digit |
|---:|---|---:|---|---:|:---:|
| `(0,1)` | `H1` | 1 | `V2` | 0 | `4` |
| `(1,4)` | `H2` | 0 | `V4` | 0 | `1` |
| `(1,6)` | `H2` | 2 | `V5` | 0 | `1` |
| `(1,8)` | `H2` | 4 | `V7` | 1 | `2` |
| `(2,0)` | `H3` | 0 | `V1` | 0 | `6` |
| `(2,1)` | `H3` | 1 | `V2` | 2 | `6` |
| `(2,2)` | `H3` | 2 | `V3` | 0 | `2` |
| `(2,4)` | `H3` | 4 | `V4` | 1 | `9` |
| `(3,4)` | `H4` | 0 | `V4` | 2 | `3` |
| `(3,6)` | `H4` | 2 | `V5` | 2 | `0` |
| `(3,8)` | `H4` | 4 | `V7` | 3 | `6` |
| `(5,0)` | `H5` | 0 | `V1` | 3 | `7` |
| `(5,2)` | `H5` | 2 | `V3` | 3 | `2` |
| `(5,4)` | `H5` | 4 | `V4` | 4 | `7` |
| `(6,4)` | `H6` | 0 | `V4` | 5 | `2` |
| `(6,6)` | `H6` | 2 | `V5` | 5 | `2` |
| `(6,7)` | `H6` | 3 | `V6` | 0 | `6` |
| `(6,8)` | `H6` | 4 | `V7` | 6 | `3` |
| `(7,0)` | `H7` | 0 | `V1` | 5 | `7` |
| `(7,2)` | `H7` | 2 | `V3` | 5 | `2` |
| `(7,4)` | `H7` | 4 | `V4` | 6 | `5` |
| `(8,7)` | `H8` | 2 | `V6` | 2 | `1` |

## Canonical solution grid

`#` cells remain blocked. Every other character becomes a `SolutionCellDefinition` entry for Hint/win verification.

```text
8423####9
#9##17142
66209#2#5
0#6#39096
1#6#1#3#3
71247#0#0
7#8#25263
79285##5#
8####6211
```

## Canonical assignment map

```text
H1 -> Level_06_P04 -> 8423
H2 -> Level_06_P05 -> 17142
H3 -> Level_06_P08 -> 66209
H4 -> Level_06_P07 -> 39096
H5 -> Level_06_P09 -> 71247
H6 -> Level_06_P06 -> 25263
H7 -> Level_06_P10 -> 79285
H8 -> Level_06_P03 -> 6211
V1 -> Level_06_P14 -> 6017778
V2 -> Level_06_P01 -> 496
V3 -> Level_06_P12 -> 266282
V4 -> Level_06_P13 -> 1931725
V5 -> Level_06_P11 -> 120302
V6 -> Level_06_P02 -> 651
V7 -> Level_06_P15 -> 9256303
```

---

# 10. Runtime derivation rules

The following data should be **derived**, not separately hand-maintained, to reduce authoring errors.

## 10.1 Blocked cells

For each character in `initialGrid`:

```text
if char == '#':
    add CellCoord(row, column) to blockedCells
```

## 10.2 Initial locked cells

```text
if char is '0'..'9':
    add LockedCellDefinition(coord, char)
```

These cells are immutable blue game-owned digits.

## 10.3 Solution cells

For each non-`#` character in `solutionGrid`:

```text
add SolutionCellDefinition(coord, digit)
```

This map is the source for single-cell Hint placement.

## 10.4 Slot cell coordinates

For a horizontal slot:

```text
cell[i] = (start.row, start.column + i)
```

For a vertical slot:

```text
cell[i] = (start.row + i, start.column)
```

for `i = 0 .. length - 1`.

## 10.5 Intersections

The serialized intersection list is optional. To derive it:

1. Expand all slots into cell coordinates.
2. Build `CellCoord -> List<(slotId, digitIndex)>`.
3. Any coordinate referenced by both a horizontal and vertical slot is an intersection.
4. Compare the two occupying piece digits at those indices during hover/placement validation.

The explicit intersection tables in this file are the expected results and can be used as unit-test fixtures.

## 10.6 Number-bank groups

Group available pieces by:

```text
piece.digits.Length
```

The six levels use lengths 3–7 only, except Level 1 which contains only 5-digit pieces.

---

# 11. Minimal editor/import validation

Before entering Play Mode, a `LevelDefinitionValidator` should reject or flag an asset when any of these checks fail:

```text
rows > 0
columns > 0
all grid rows have exactly columns characters
slotId values are unique
pieceId values are unique
all slot lengths are 3..7
piece.digits.Length matches its assigned/tested slot length
all slot cells are inside the grid
no slot cell is blocked
all active grid cells belong to >= 1 slot
all initial locked digits equal solutionGrid at the same cell
every solution active cell has exactly one final digit
all crossing canonical digits agree
piece count == slot count for these six levels
canonical piece set == bank piece set
```

For this case-study dataset, expected level validation totals are:

```text
Level_01: 2 slots,  1 intersection,  2 initial locked cells
Level_02: 6 slots, 6 intersections, 12 initial locked cells
Level_03: 6 slots, 6 intersections, 12 initial locked cells
Level_04: 10 slots, 10 intersections, 13 initial locked cells
Level_05: 10 slots, 10 intersections, 14 initial locked cells
Level_06: 15 slots, 22 intersections, 11 initial locked cells
```

If generated runtime counts do not match these numbers, the level data was transcribed incorrectly.

---

# 12. Important gameplay boundary

This file supplies **level content only**. Placement, hover colors, eviction, replacement, hints, persistence, progression, and win behavior must follow `GAME_SPEC.md` and `IMPLEMENTATION_SPEC.md`.

In particular:

- same-length slots are candidates;
- matching hover is green;
- conflicting hover is red/pink;
- a player-player contradiction may still be committed and evicts the conflicting older player piece;
- replacing an occupied slot returns the old piece to the bank;
- locked blue digits cannot be removed;
- Hint fills one empty solution cell as a new locked blue digit;
- there is no fail condition;
- gameplay does not change between Levels 1–6.

---

# 13. Digit-class lookup for UI interaction

The digit-class/slot-highlighting mechanic requires **no additional authored geometry**. Derive digit classes directly from `SlotDefinition.length` and bank classes from `PieceDefinition.digits.Length`.

Expected classes and slot counts:

| Level | 3 digits | 4 digits | 5 digits | 6 digits | 7 digits |
|---|---:|---:|---:|---:|---:|
| Level 1 | 0 | 0 | 2 | 0 | 0 |
| Level 2 | 2 | 0 | 0 | 4 | 0 |
| Level 3 | 2 | 0 | 0 | 4 | 0 |
| Level 4 | 2 | 2 | 2 | 2 | 2 |
| Level 5 | 2 | 2 | 2 | 2 | 2 |
| Level 6 | 2 | 2 | 6 | 2 | 3 |

At runtime build:

```text
slotsByLength: Dictionary<int, List<SlotDefinition>>
slotsByCell:   Dictionary<CellCoord, List<SlotDefinition>>
bankGroupByLength: Dictionary<int, BankGroupView>
```

For a non-crossing cell, `slotsByCell[cell]` contains one slot. At an intersection it contains the horizontal and vertical slots. The vertical-first repeated-press cycling behavior is defined in `GAME_SPEC.md` / `IMPLEMENTATION_SPEC.md`; do not encode it in level assets.

