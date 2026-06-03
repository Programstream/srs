# SRS-PRO — Software Requirements Specification
## Program — Master Calendar Generator
### Version 2

---

## Project Overview

**Program** is a Python CLI tool that generates a master programme calendar for 2026–2028, outputting a single `.xlsx` file using `openpyxl`. It is a personal planning tool for a solo practitioner running a programme of SaaS, marketing, and other project work on a repeating 17-week cycle blueprint.

The current output file is `Master_Program_2026_2028_v1.xlsx`. Study it carefully before making any changes — it is the ground truth for layout, structure, and color.

---

## What You Are Being Asked to Implement

Three related changes to the existing calendar generator:

1. **Add a new `PRE` row** to each calendar band — sitting above the existing `PROG` row — showing the preparation stream phases
2. **Replace the existing `PROG` row** phase codes (`R`, `S`, `G`) with the new POST delivery cycle phases
3. **Update YAML configuration** to support the new two-stream blueprint structure

The calendar will now show **two parallel streams** for every day in the 2026–2028 range:
- `PRE` — the preparation stream (what is being built toward the next Freeze)
- `POST` — the delivery stream (what is being tested and released)

Both streams run on the same 17-week repeating cycle, offset so that PRE for the next cycle overlaps with POST of the current cycle. This is intentional and correct — while one thing is being tested/released, the next thing is already being built.

---

## Existing Codebase — What You Must Understand First

Before writing any code, read every Python source file in this repository. The key things to locate and understand are:

### Configuration files (YAML)
- `holidays.yaml` — defines all public holidays (Swiss / Canton Zurich), school holidays (ISCS Cham), and personal leave dates used to populate the `HOL` row
- `master_program.yaml` — defines the programme structure: stream configuration, phase assignments, capacity buffers, and the output filename pattern

### Source modules to locate
Find and read the module responsible for:
- Reading YAML config
- Computing which days get which phase code
- Writing the Excel file via `openpyxl`

### Existing holiday and constraint logic — DO NOT MODIFY
The `HOL` row is populated with these codes and colors, computed automatically from the YAML config. Leave this logic entirely untouched:

| Code | Meaning | Background Color (hex) |
|---|---|---|
| `H` | Swiss public holiday (Canton Zurich) | `FFFFB3B3` (pastel red) |
| `SH` | School holiday (ISCS Cham) | `FFB3D9FF` (pastel blue) |
| `PL` | Personal leave | `FFFFD9B3` (pastel orange) |

### Existing weekend logic — DO NOT MODIFY
Saturdays and Sundays in the `WD` row are colored `FFD9D9D9` (light gray). Phase rows (`PRE`, `POST`) on weekends still show the phase code and color — weekends do not advance the phase calendar-week counter, but the phase is still displayed.

### Existing capacity buffer logic — PRESERVE
The tool currently skips phase assignment on holidays and leave days. For the new implementation: **phase codes are always shown** (including on weekends and holidays) but **calendar weeks are counted in full weeks only** — a week that contains a public holiday still counts as one calendar week. Do not skip cells; always render the phase.

---

## Excel Layout — Exact Existing Structure

The output workbook has a single sheet called `Calendar`. It shows **two months side by side** per row band:

- **Columns**: A (row labels) + columns B–AF (month 1, days 1–31) + column AG (separator, empty) + columns AH–BL (month 2, days 1–31). Total: 69 columns.
- **Row bands**: 18 bands total (Jan/Feb 2026, Mar/Apr 2026 … Nov/Dec 2028)
- **Band height**: 6 rows with a 1-row spacer between bands

### Existing row structure per band:

| Offset | Label | Content |
|---|---|---|
| +0 | *(blank)* | Month name headers |
| +1 | `Dat` | Day numbers 1–31 |
| +2 | `WD` | Weekday abbreviations |
| +3 | `HOL` | Holiday/leave codes |
| +4 | `PROG` | Current phase codes (R, S, G) — **being replaced** |
| +5 | *(blank)* | Spacer |

**Band start rows**: `1, 8, 14, 20, 26, 32, 38, 44, 50, 56, 62, 68, 74, 80, 86, 92, 98, 104`

**Important exception**: Band 1 (rows 1–7) has an extra blank row between HOL and PROG — PROG is at row 6, not row 5. All other bands follow standard offset.

---

## New Row Structure Per Band

Add one new row (`PRE`) between `HOL` and `POST`. The existing `PROG` row is renamed to `POST`. The band height increases from 6 to 7 rows. Update all band spacing accordingly.

### New row structure per band:

| Offset | Label | Content |
|---|---|---|
| +0 | *(blank)* | Month name headers |
| +1 | `Dat` | Day numbers 1–31 |
| +2 | `WD` | Weekday abbreviations |
| +3 | `HOL` | Holiday/leave codes — unchanged |
| +4 | `PRE` | PRE stream phase codes — **NEW** |
| +5 | `POST` | POST stream phase codes — **replaces PROG** |
| +6 | *(blank)* | Spacer |

---

## The Blueprint Cycle — 17 Calendar Weeks, 3 Cycles Per Year

The entire programme runs on a single repeating 17-week blueprint cycle. There are no project-specific streams visible in the calendar — the calendar shows one unified programme rhythm. Both PRE and POST rows display this same cycle, offset from each other.

### PRE stream phases (8 calendar weeks + 1 week break = 9 weeks total):

| Code | Full Name | Duration | Cycle weeks |
|---|---|---|---|
| `DEF` | Define | 2 calendar weeks | Weeks 1–2 |
| `SCP` | Scope | 2 calendar weeks | Weeks 3–4 |
| `BLD` | Build | 3 calendar weeks | Weeks 5–7 |
| `BRK` | Break | 1 calendar week | Week 8 |

**PRE total: 8 weeks active + 1 week break = 9 weeks**

### POST stream phases (1 day + ~8 weeks active + 1 week break = ~9 weeks total):

| Code | Full Name | Duration | Cycle weeks |
|---|---|---|---|
| `FRE` | Freeze | 1 calendar day | Week 9 day 1 |
| `VER` | Verify | 2 calendar weeks | Weeks 9–10 |
| `VAL` | Validate | 3 calendar weeks | Weeks 11–13 |
| `REL` | Release | 2 calendar weeks | Weeks 14–15 |
| `PCO` | Post Cut Over | 1 calendar week | Week 16 |
| `BRK` | Break | 1 calendar week | Week 17 |

**POST total: ~8 weeks active + 1 week break = ~9 weeks**

**Full cycle: 17 calendar weeks → 3.05 cycles per year → exactly 3 cycles**

### Key facts about the cycle:

- All durations are **calendar weeks**, not working days. A week with a public holiday in it still counts as one full week. The phase advances on the Monday of the next week regardless of holidays within it.
- `FRE` is the only exception — it is exactly **1 calendar day**, always a Monday (the first day of week 9).
- `BRK` appears in **both streams** using the same code. In PRE it falls at week 8. In POST it falls at week 17. Both use the same gray color.
- The cycle repeats immediately after POST `BRK` ends — the next PRE `DEF` begins the following Monday.
- **PRE and POST are offset by 8 weeks** — when POST starts its FRE (week 9 of the POST cycle), PRE is already in week 1 of the next cycle's DEF. This overlap is correct and intentional.

### 3 cycles per year across 2026–2028:

With a 17-week cycle and a programme start of January 2026, the approximate cycle boundaries are:

| Cycle | PRE starts | POST FRE | POST BRK ends |
|---|---|---|---|
| 2026 Cycle 1 | Jan 5 2026 | Mar 2 2026 | Jun 28 2026 |
| 2026 Cycle 2 | Apr 27 2026 | Jun 22 2026 | Oct 18 2026 |
| 2026 Cycle 3 | Aug 17 2026 | Oct 12 2026 | Feb 7 2027 |
| 2027 Cycle 1 | Dec 7 2026 | Feb 1 2027 | May 30 2027 |
| *(continues…)* | | | |

The exact dates are computed from the configured `programme_start` date in YAML — do not hardcode them. The dates above are illustrative only.

---

## Color Scheme

### PRE stream colors:

| Code | Full Name | Color Name | Hex (openpyxl fgColor) |
|---|---|---|---|
| `DEF` | Define | Soft lavender | `FFE8EAF6` |
| `SCP` | Scope | Medium lavender | `FFC5CAE9` |
| `BLD` | Build | Deep lavender | `FF9FA8DA` |
| `BRK` | Break | Light gray | `FFE0E0E0` |

### POST stream colors:

| Code | Full Name | Color Name | Hex (openpyxl fgColor) |
|---|---|---|---|
| `FRE` | Freeze | Deep slate blue | `FF5C6BC0` |
| `VER` | Verify | Pastel teal | `FFB2DFDB` |
| `VAL` | Validate | Pastel yellow | `FFFFF9C4` |
| `REL` | Release | Pastel green | `FFC8E6C9` |
| `PCO` | Post Cut Over | Pastel peach | `FFFFE0B2` |
| `BRK` | Break | Light gray | `FFE0E0E0` |

### Design rationale:
- PRE phases use a lavender progression (light → medium → deep) showing build-up toward Freeze
- POST phases use distinct colors per phase for clear visual differentiation during testing/release
- Both `BRK` codes share the same gray — they are the same concept in both streams
- `FRE` is the darkest color in the palette — it is a single day and must stand out visually
- All colors are harmonious with the existing HOL palette (H=`FFFFB3B3`, SH=`FFB3D9FF`, PL=`FFFFD9B3`)

Apply colors using `openpyxl.styles.PatternFill(fill_type="solid", fgColor="...")` to every cell in the PRE and POST rows, including weekends and holidays.

---

## YAML Configuration Changes Required

Replace the existing stream/phase configuration with the following structure. The key new field is `programme_start` — the Monday on which the very first PRE `DEF` phase begins:

```yaml
programme:
  start_date: "2026-01-05"      # Monday — first day of PRE DEF cycle 1
  output_name: "Master_Program_2026_2028"

cycle:
  pre:
    - code: DEF
      weeks: 2
    - code: SCP
      weeks: 2
    - code: BLD
      weeks: 3
    - code: BRK
      weeks: 1
  post:
    - code: FRE
      days: 1                   # FRE is 1 calendar day, not weeks
    - code: VER
      weeks: 2
    - code: VAL
      weeks: 3
    - code: REL
      weeks: 2
    - code: PCO
      weeks: 1
    - code: BRK
      weeks: 1
```

The `start_date` is a placeholder — the project owner will supply the correct date. It must always be a **Monday**. Add a validation check that raises a clear error if `start_date` falls on a non-Monday.

Do not hardcode any dates, durations, or phase names anywhere in the Python logic. Everything comes from YAML.

---

## Implementation Approach

### Step 1 — Read and map the existing code
Before writing a single line, read all source files. Run the existing tool and confirm the output matches `Master_Program_2026_2028_v1.xlsx`. Understand exactly where phase codes are written to cells.

### Step 2 — Implement `cycle_engine.py`
Create a new module with two public functions:

```python
def get_pre_phase_for_date(
    target_date: date,
    programme_start: date,
    cycle_config: dict
) -> str | None:
    """
    Returns the PRE stream phase code for the given date.
    Returns one of: 'DEF', 'SCP', 'BLD', 'BRK'
    Returns None if date is before programme_start.
    Phase boundaries are computed in calendar weeks from programme_start.
    """

def get_post_phase_for_date(
    target_date: date,
    programme_start: date,
    cycle_config: dict
) -> str | None:
    """
    Returns the POST stream phase code for the given date.
    Returns one of: 'FRE', 'VER', 'VAL', 'REL', 'PCO', 'BRK'
    Returns None if date is before the first FRE day.
    FRE is exactly 1 calendar day (the Monday of week 9 of each cycle).
    All other POST phases are measured in calendar weeks.
    POST is offset from PRE by the total PRE cycle length (9 weeks).
    """
```

Both functions must:
- Work purely from `programme_start` and `cycle_config` — no hardcoded dates
- Return a phase code for every calendar date including weekends and holidays
- Handle the repeating cycle correctly across the full 2026–2028 range

### Step 3 — Update Excel band structure
Insert the new `PRE` row at offset +4 in every band. Shift the existing `PROG` row (now `POST`) to offset +5. Update the spacer to offset +6. Handle the Band 1 exception (extra blank row). Update all row index references throughout the codebase.

### Step 4 — Wire phase functions into Excel writer
Replace the existing phase-writing logic with calls to `cycle_engine.get_pre_phase_for_date()` and `cycle_engine.get_post_phase_for_date()`. Write both PRE and POST rows for every band.

### Step 5 — Apply colors
Apply the color tables above to PRE and POST rows. Remove old R/S/G color mappings.

### Step 6 — Update YAML schema and reader
Implement the new YAML structure. Add Monday validation for `start_date`.

---

## Testing Requirements

After implementation, verify all of the following:

**Cycle structure:**
1. PRE `DEF` begins on `programme_start` (a Monday) and spans exactly 2 calendar weeks
2. PRE `SCP` follows immediately for exactly 2 calendar weeks
3. PRE `BLD` follows immediately for exactly 3 calendar weeks
4. PRE `BRK` follows immediately for exactly 1 calendar week
5. POST `FRE` falls on the Monday immediately after PRE `BRK` ends — exactly 1 day
6. POST `VER` follows `FRE` for exactly 2 calendar weeks
7. POST `VAL` follows for exactly 3 calendar weeks
8. POST `REL` follows for exactly 2 calendar weeks
9. POST `PCO` follows for exactly 1 calendar week
10. POST `BRK` follows for exactly 1 calendar week
11. The next cycle's PRE `DEF` begins the Monday after POST `BRK` ends
12. Spot-check: verify at least 3 full cycle repetitions are correct across the 2026–2028 range

**Holiday behavior:**
13. A week containing a Swiss public holiday still advances the phase to the next week on schedule — holidays do not extend phase durations
14. PRE and POST cells on public holiday days still show their phase code and color

**Layout:**
15. Every band now has 7 rows (header + Dat + WD + HOL + PRE + POST + spacer)
16. Band 1 exception is handled correctly
17. `HOL`, `Dat`, `WD` rows are byte-for-byte identical to the baseline output
18. Column structure is unchanged (69 columns, 2 months side by side)

**Colors:**
19. All PRE and POST cells display correct hex colors per the color table
20. Both `BRK` rows (PRE and POST) display the same `FFE0E0E0` gray
21. `FRE` day displays `FF5C6BC0` and is visually prominent as a single-day marker

**Version:**
22. Output filename auto-increments correctly (v2 if v1 exists, etc.)

---

## Hard Rules — Do Not Violate

- **Never modify `HOL` row logic.** Holiday and leave computation is correct and must not be touched.
- **Never hardcode dates or durations.** Everything comes from YAML.
- **Never change the Excel column structure.** 69 columns, 2 months side by side, separator at column 33/34.
- **Always render phase codes on weekends and holidays.** No blank cells in PRE or POST rows.
- **Phase durations are calendar weeks.** Holidays within a week do not extend the phase.
- **`FRE` is always exactly 1 calendar day** — never stretch it to fill a week.
- **`programme_start` must be a Monday.** Validate and raise a clear error if not.
- **Preserve version auto-increment behaviour.**
- **Do not modify any existing test files** without reading them first and confirming the change is necessary.

---

## Reference: What Is Being Replaced

| Old Code | Old Color | Stream | Replaced By |
|---|---|---|---|
| `R` (Ready) | `FFFADADD` | PROG | PRE: `DEF`, `SCP`, `BLD` |
| `S` (Steady) | `FFFFF9C4` | PROG | POST: `VER`, `VAL` |
| `G` (Go/No-Go) | `FFC8E6C9` | PROG | POST: `REL` |
| *(none)* | — | *(none)* | POST: `FRE`, `PCO`, `BRK` — new |
| *(none)* | — | *(none)* | PRE: `BRK` — new |

---

## Questions to Raise Before Proceeding

If any of the following are unclear after reading the source code, stop and ask the project owner before writing any code:

1. What is the exact CLI command to run the tool?
2. What is the correct `programme_start` date for the 2026 calendar? (Must be a Monday — suggest 2026-01-05 but confirm.)
3. Where exactly in the existing code are phase codes written to Excel cells? Show the project owner the function name and file before modifying it.
4. Does the existing code have any unit tests? If yes, read them before changing anything.
5. Are there any other output files or sheets in the workbook besides `Calendar`?
