# SRS-PRO — Software Requirements Specification
## Program — Master Calendar Generator
### Version 4

---

## What Changed in v4

Five precise changes. Everything not listed here remains as specified in v2 and v3.

1. Stream row labels renamed: `PRE` → `BLD`, `POST` → `TEST`
2. Phase codes renamed: `REL` → `REA`, `PCO` → `PGL`
3. End-of-phase review days added to both streams
4. `HDV` and `BRK` cells: white base with faint amber diagonal stripe, no text
5. Darkening function specified for review day colors

---

## Change 1 — Stream Row Labels

The two calendar rows are now labelled:

| Old label | New label | Meaning |
|---|---|---|
| `PRE` | `BLD` | Build stream — preparation toward Freeze |
| `POST` | `TEST` | Test stream — delivery from Freeze to release |

Update column A labels in the Excel output accordingly. Update all variable names, function names, YAML keys, and comments in the codebase to use `bld` and `test` consistently. Do not leave any reference to `pre` or `post` as stream identifiers.

---

## Change 2 — Phase Code Renames

Two phase codes in the TEST stream are renamed:

| Old code | New code | Full name | Stream |
|---|---|---|---|
| `REL` | `REA` | Ready | TEST |
| `PCO` | `PGL` | Post Go Live | TEST |

No duration, color, or position changes. Only the code and label change. Update YAML, cycle engine, color mapping, and any legend or documentation.

---

## Change 3 — End-of-Phase Review Days

### Concept

The **last calendar day** of each phase (except `FRE`, `HDV`, and `BRK`) is replaced by a **review day**. The review day signals the closing beat of the phase — a moment to wrap up, review, and confirm readiness before the next phase begins.

### Review day codes

Each review day uses the phase code with `R` appended:

| Phase | Review code | Row label shown in cell (2 lines) | Stream |
|---|---|---|---|
| `DEF` | `DEFR` | line 1: `DEF` / line 2: `REV` | BLD |
| `SCP` | `SCPR` | line 1: `SCP` / line 2: `REV` | BLD |
| `BLD` | `BLDR` | line 1: `BLD` / line 2: `REV` | BLD |
| `VER` | `VERR` | line 1: `VER` / line 2: `REV` | TEST |
| `VAL` | `VALR` | line 1: `VAL` / line 2: `REV` | TEST |
| `REA` | `REAR` | line 1: `REA` / line 2: `REV` | TEST |
| `PGL` | `PGLR` | line 1: `PGL` / line 2: `REV` | TEST |

**`FRE` has no review day** — it is already 1 calendar day and is the pivot itself.
**`HDV` has no review day** — it is a transition period, not an active phase.
**`BRK` has no review day** — it is a rest period, not an active phase.

### Cell rendering for review days

Each review day cell must display two lines of text in the same cell:

```python
cell.value = "DEF\nREV"   # example for DEFR
cell.alignment = openpyxl.styles.Alignment(
    wrap_text=True,
    horizontal="center",
    vertical="center"
)
```

Apply this two-line wrapped rendering to all review day cells. All other phase cells remain single-line as before.

### Review day color — 30% darker than phase color

Review day cells use a **programmatically darkened** version of their phase color. Implement the following function in `cycle_engine.py`:

```python
def darken_hex(hex_color: str, factor: float = 0.70) -> str:
    """
    Darkens an openpyxl hex color string by multiplying each RGB channel
    by factor (0.70 = 30% darker).

    Input format: 'FFRRGGBB' (openpyxl fgColor format, with FF alpha prefix)
    Output format: 'FFRRGGBB'

    Example:
        darken_hex('FFE8EAF6', 0.70) → 'FFA2A4AC'
    """
    r = int(hex_color[2:4], 16)
    g = int(hex_color[4:6], 16)
    b = int(hex_color[6:8], 16)
    r = int(r * factor)
    g = int(g * factor)
    b = int(b * factor)
    return f"FF{r:02X}{g:02X}{b:02X}"
```

Call this function at color assignment time — do not precompute and hardcode the darkened hex values. The base phase colors remain the source of truth; darkened values are always derived.

### Review day placement rule

The review day is always the **last calendar day** of the phase — including if that day falls on a weekend or holiday. The phase duration does not change. The review day consumes the last day of the phase's calendar week allocation.

Example for `DEF` (2 calendar weeks = 14 calendar days):
- Days 1–13: `DEF` (base color)
- Day 14: `DEFR` (30% darker, two-line text)

Example for `BLD` (3 calendar weeks = 21 calendar days):
- Days 1–20: `BLD` (base color)
- Day 21: `BLDR` (30% darker, two-line text)

---

## Change 4 — HDV and BRK: Diagonal Stripe, No Text

`HDV` (BLD stream, week 8) and `BRK` (TEST stream, week 17) are transition/rest periods. They must be visually empty — no active phase color, no text — but distinguishable from blank cells.

### Rendering specification

```python
from openpyxl.styles import PatternFill

stripe_fill = PatternFill(
    patternType="darkUp",       # diagonal stripe pattern
    fgColor="FFFFF8E1",         # faint warm amber stripe
    bgColor="FFFFFFFF"          # white base
)

cell.value = None               # empty — no text, no code
cell.fill = stripe_fill
```

Apply this to **every cell** in the `HDV` week (BLD row) and `BRK` week (TEST row), including weekends and holidays within those weeks.

### What this looks like
- Base: white
- Stripe: barely-visible warm amber diagonal lines
- Text: none
- Effect: reads as "empty / resting" while still hinting at the transition amber from v3

### Hard rule
Do not write any cell value (`HDV`, `BRK`, or any other string) to these cells. The value must be `None`. The only thing applied is the `PatternFill`.

---

## Change 5 — Darkening Function

Specified in Change 3 above. Summary:

- Function name: `darken_hex`
- Location: `cycle_engine.py`
- Factor: `0.70` (30% darker)
- Called at runtime during color assignment — never precomputed into hardcoded hex strings
- Applied to: `DEFR`, `SCPR`, `BLDR`, `VERR`, `VALR`, `REAR`, `PGLR` cells only

---

## Full Updated Phase Tables

### BLD stream (formerly PRE):

| Code | Full Name | Duration | Cycle weeks | Base color hex | Review code |
|---|---|---|---|---|---|
| `DEF` | Define | 2 weeks | Weeks 1–2 | `FFE8EAF6` | `DEFR` on day 14 |
| `SCP` | Scope | 2 weeks | Weeks 3–4 | `FFC5CAE9` | `SCPR` on day 28 |
| `BLD` | Build | 3 weeks | Weeks 5–7 | `FF9FA8DA` | `BLDR` on day 49 |
| `HDV` | Hand Over | 1 week | Week 8 | — stripe only — | none |

### TEST stream (formerly POST):

| Code | Full Name | Duration | Cycle weeks | Base color hex | Review code |
|---|---|---|---|---|---|
| `FRE` | Freeze | 1 day | Week 9 day 1 | `FF5C6BC0` | none |
| `VER` | Verify | 2 weeks | Weeks 9–10 | `FFB2DFDB` | `VERR` on last day |
| `VAL` | Validate | 3 weeks | Weeks 11–13 | `FFFFF9C4` | `VALR` on last day |
| `REA` | Ready | 2 weeks | Weeks 14–15 | `FFC8E6C9` | `REAR` on last day |
| `PGL` | Post Go Live | 1 week | Week 16 | `FFFFE0B2` | `PGLR` on last day |
| `BRK` | Break | 1 week | Week 17 | — stripe only — | none |

---

## Full Updated Color Reference

### Base phase colors (source of truth):

| Stream | Code | Hex |
|---|---|---|
| BLD | `DEF` | `FFE8EAF6` |
| BLD | `SCP` | `FFC5CAE9` |
| BLD | `BLD` | `FF9FA8DA` |
| TEST | `FRE` | `FF5C6BC0` |
| TEST | `VER` | `FFB2DFDB` |
| TEST | `VAL` | `FFFFF9C4` |
| TEST | `REA` | `FFC8E6C9` |
| TEST | `PGL` | `FFFFE0B2` |

### Review day colors (computed via darken_hex at runtime, never hardcoded):

| Code | Derived from | Applied to |
|---|---|---|
| `DEFR` | `darken_hex('FFE8EAF6')` | Last day of DEF |
| `SCPR` | `darken_hex('FFC5CAE9')` | Last day of SCP |
| `BLDR` | `darken_hex('FF9FA8DA')` | Last day of BLD |
| `VERR` | `darken_hex('FFB2DFDB')` | Last day of VER |
| `VALR` | `darken_hex('FFFFF9C4')` | Last day of VAL |
| `REAR` | `darken_hex('FFC8E6C9')` | Last day of REA |
| `PGLR` | `darken_hex('FFFFE0B2')` | Last day of PGL |

### Transition/rest fill (PatternFill, not solid):

| Code | Pattern | fgColor | bgColor | Cell value |
|---|---|---|---|---|
| `HDV` | `darkUp` | `FFFFF8E1` | `FFFFFFFF` | `None` |
| `BRK` | `darkUp` | `FFFFF8E1` | `FFFFFFFF` | `None` |

### Reserved colors (do not use in BLD or TEST rows):

| Hex | Reserved for |
|---|---|
| `FFE0E0E0` | `WD` row weekends only |
| `FFFFB3B3` | `HOL` row — public holidays |
| `FFB3D9FF` | `HOL` row — school holidays |
| `FFFFD9B3` | `HOL` row — personal leave |

---

## Updated YAML Configuration

```yaml
cycle:
  bld:                          # formerly: pre
    - code: DEF
      weeks: 2
    - code: SCP
      weeks: 2
    - code: BLD
      weeks: 3
    - code: HDV
      weeks: 1
      style: stripe             # signals: no solid color, use PatternFill
  test:                         # formerly: post
    - code: FRE
      days: 1
    - code: VER
      weeks: 2
    - code: VAL
      weeks: 3
    - code: REA
      weeks: 2
    - code: PGL
      weeks: 1
    - code: BRK
      weeks: 1
      style: stripe             # signals: no solid color, use PatternFill
```

The `style: stripe` flag tells the rendering engine to apply `PatternFill` instead of `PatternFill(fill_type="solid")` for that phase. All phases without `style: stripe` use solid fill.

---

## Updated Testing Checklist (v4 additions)

Add these test items to the v2/v3 checklist:

**Stream labels:**
- T-24: Column A shows `BLD` and `TEST` as row labels. No cell in column A reads `PRE` or `POST`.

**Phase renames:**
- T-25: No cell in the TEST row contains `REL` or `PCO` anywhere in the 2026–2028 output.
- T-26: `REA` and `PGL` appear in the correct positions within each TEST cycle.

**Review days:**
- T-27: The last day of each `DEF` phase shows code `DEFR`, two-line text `DEF\nREV`, wrap_text=True, color = `darken_hex('FFE8EAF6', 0.70)`.
- T-28: Verify the same for `SCPR`, `BLDR`, `VERR`, `VALR`, `REAR`, `PGLR` in their respective phases.
- T-29: `FRE`, `HDV`, and `BRK` have no review day cell adjacent to them.
- T-30: Review day cells on weekends or holidays still show the darkened color and two-line text — they are not skipped.

**Stripe cells:**
- T-31: Every cell in an `HDV` week (BLD row) has `cell.value = None`, `patternType="darkUp"`, `fgColor="FFFFF8E1"`, `bgColor="FFFFFFFF"`.
- T-32: Every cell in a `BRK` week (TEST row) has the same stripe fill and `cell.value = None`.
- T-33: No cell in the BLD or TEST rows has fill color `FFE0E0E0`.

**Darkening function:**
- T-34: Call `darken_hex('FFE8EAF6', 0.70)` and confirm output is `FFA2A4AD` (±1 rounding). Implement as a unit test.
- T-35: Confirm `darken_hex` is never called with a hardcoded review hex — it always receives the base phase color as input.

---

## Hard Rules Added in v4

In addition to all hard rules from v2 and v3:

- **Row labels in column A are `BLD` and `TEST`.** Never write `PRE` or `POST` to column A.
- **`REA` and `PGL` are the correct codes.** Never write `REL` or `PCO` to any cell.
- **Review day cells must use `wrap_text=True`.** A single-line cell is not acceptable for review days.
- **`darken_hex` must not be bypassed.** Never hardcode a darkened hex value — always derive it from the base color at runtime.
- **`HDV` and `BRK` cells must have `cell.value = None`.** Writing any string to these cells is incorrect.
- **`PatternFill` stripe is not the same as solid fill.** Do not apply `fill_type="solid"` to `HDV` or `BRK` cells.

---

## No Other Changes

Everything not listed in this document remains as specified in v2 and v3:
- Excel band structure and row offsets — v2
- Cycle timing and week counts — v2
- Programme start date and Monday validation — v2
- Implementation steps 1–6 — v2
- Visual pivot column alignment (HDV / FRE) — v3

Refer to SRS-PRO-v2 and SRS-PRO-v3 for all sections not covered here.
