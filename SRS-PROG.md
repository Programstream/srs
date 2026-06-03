# Program — Master Calendar Generator
## Claude Code Project Instructions

---

## Project Overview

**Program** is a Python CLI tool that generates a multi-project master programme calendar for 2026–2028, outputting a single `.xlsx` file using `openpyxl`. It is a personal planning tool for a solo practitioner running multiple SaaS and marketing project streams simultaneously.

The current output file is `Master_Program_2026_2028_v1.xlsx`. Study it carefully before making any changes — it is the ground truth for layout, structure, and color.

---

## What You Are Being Asked to Implement

Two related changes to the existing calendar generator:

1. **Rename the RSG phases** — the existing phase codes `R`, `S`, `G` in the `PROG` row are being replaced with a new 6-phase delivery cycle
2. **Implement the new 6-phase cycle structure** — the new phases must be computed from configured stream start dates and rendered in the `PROG` row with correct codes, colors, and durations

Do not touch anything else. All other rows (`Dat`, `WD`, `HOL`) and all layout/color logic unrelated to `PROG` must remain byte-for-byte identical.

---

## Existing Codebase — What You Must Understand First

Before writing any code, read every Python source file in this repository. The key things to locate and understand are:

### Configuration files (YAML)
- `holidays.yaml` — defines all public holidays (Swiss / Canton Zurich), school holidays (ISCS Cham), and personal leave dates used to populate the `HOL` row
- `master_program.yaml` — defines the programme structure: project streams, phase assignments, capacity buffers, and the output filename pattern

### Source modules to locate
Find and read the module responsible for:
- Reading YAML config
- Computing which days get which phase code in `PROG`
- Writing the Excel file via `openpyxl`

### Existing holiday and constraint logic — DO NOT MODIFY
The `HOL` row is populated with these codes and colors. They are computed automatically from the YAML config. Leave this logic entirely untouched:

| Code | Meaning | Background Color (hex) |
|---|---|---|
| `H` | Swiss public holiday (Canton Zurich) | `FFFFB3B3` (pastel red) |
| `SH` | School holiday (ISCS Cham) | `FFB3D9FF` (pastel blue) |
| `PL` | Personal leave | `FFFFD9B3` (pastel orange) |

### Existing weekend logic — DO NOT MODIFY
Saturdays and Sundays in the `WD` row are colored `FFD9D9D9` (light gray). `PROG` cells on weekends are left empty (no phase code assigned). This logic must be preserved exactly.

### Existing capacity buffer logic — DO NOT MODIFY
The tool skips phase assignment on days that are holidays (`H`, `SH`) or personal leave (`PL`). These days do not count toward phase duration. Preserve this buffer logic exactly — the new phases must respect it in the same way the old `R`/`S`/`G` phases did.

---

## Excel Layout — Exact Structure

The output workbook has a single sheet called `Calendar`. It shows **two months side by side** per row band. The layout is:

- **Columns**: A (row labels) + columns B–AF (month 1, days 1–31) + column AG (separator, empty) + columns AH–BL (month 2, days 1–31). Total: 69 columns used.
- **Row bands**: 18 bands total (one per month pair: Jan/Feb 2026, Mar/Apr 2026 … Nov/Dec 2028)
- **Band height**: 6 rows (rows 1–6 for band 1; rows 8–13 for band 2; rows 14–19 for band 3; etc.) with a 1-row spacer between most bands

### Row structure within each band (offsets from band start row):

| Offset | Column A label | Content |
|---|---|---|
| +0 | *(blank)* | Month name header (e.g. "January 2026" in col B, "February 2026" in col AH) |
| +1 | `Dat` | Day numbers 1–31 for each month |
| +2 | `WD` | Weekday abbreviations (Mon/Tue/Wed/Thu/Fri/Sat/Sun) |
| +3 | `HOL` | Holiday/leave codes (H, SH, PL) — auto-computed, do not touch |
| +4 | `PROG` | **Phase codes — this is what you are changing** |
| +5 | *(blank)* | Spacer row before next band |

The `PROG` row is row offset +4 from each band's start row. Band start rows are:
`1, 8, 14, 20, 26, 32, 38, 44, 50, 56, 62, 68, 74, 80, 86, 92, 98, 104`

So `PROG` rows are: `5, 12, 18, 24, 30, 36, 42, 48, 54, 60, 66, 72, 78, 84, 90, 96, 102, 108`

**Important exception**: Band 1 (rows 1–7) has an extra blank row at offset +4 between HOL and PROG, making PROG at row 6 (offset +5) with a spacer at row 5. All other bands follow the standard offset above.

---

## New Phase Structure — The 6-Phase Delivery Cycle

Replace the old `R` / `S` / `G` phase codes with the following 6 phases. These are applied per project stream, sequentially, repeating every ~104 working days.

### Phase definitions

| Code | Full Name | Duration | Notes |
|---|---|---|---|
| `FRE` | Freeze | **1 working day** | Code/campaign freeze — functional prototype is locked |
| `VER` | Verify | **2 weeks** (10 working days) | Internal testing — team validates |
| `VAL` | Validate | **remainder of 90 days** (see calculation below) | External/user validation |
| `REL` | Release | **2 weeks** (10 working days) | Final confidence check and go-live |
| `PCO` | Post Cut Over | **2 weeks** (10 working days) | Stabilisation, fixes, monitoring |
| `BRK` | Break | **1 week** (5 working days) | Rest period before next cycle |

### Duration calculation

The 90-day window covers `FRE + VER + VAL + REL` only:
- FRE = 1 working day
- VER = 10 working days
- REL = 10 working days
- VAL = 90 − 1 − 10 − 10 = **69 working days**

After the 90-day block:
- PCO = 10 working days (outside 90 days)
- BRK = 5 working days (outside 90 days)

**Total cycle = 104 working days**, then repeats with a new `FRE`.

### Working day counting rules

All phase durations are counted in **working days only**. A working day is any day that is:
- Monday through Friday, AND
- NOT a public holiday (`H`), AND
- NOT a school holiday (`SH`), AND
- NOT a personal leave day (`PL`)

Weekends, holidays, and leave days do not advance the phase counter. The phase code is still written to those cells (so the calendar shows continuous phase coverage), but they do not consume the phase's working day budget.

---

## Color Scheme for New Phases

Apply the following background colors to `PROG` row cells. Use `openpyxl.styles.PatternFill` with `fill_type="solid"`:

| Code | Color Name | Hex (openpyxl fgColor) |
|---|---|---|
| `FRE` | Deep slate blue | `FF5C6BC0` |
| `VER` | Pastel teal | `FFB2DFDB` |
| `VAL` | Pastel yellow | `FFFFF9C4` |
| `REL` | Pastel green | `FFC8E6C9` |
| `PCO` | Pastel peach | `FFFFE0B2` |
| `BRK` | Light gray | `FFE0E0E0` |

Weekend and holiday cells in the `PROG` row: apply the phase color (the phase is still shown), but leave the cell value as the phase code. The visual distinction for non-working days is already handled by the `WD` row (gray background) and `HOL` row.

---

## YAML Configuration Changes Required

Add the following to `master_program.yaml` (or wherever stream phase configuration lives). Each stream needs a `cycle_start` date — the first `FRE` day of the stream's first cycle:

```yaml
streams:
  - name: FAB
    cycle_start: "2026-01-05"   # First FRE day for this stream
  - name: FLA
    cycle_start: "2026-01-05"
  - name: ORC
    cycle_start: "2026-04-01"
  - name: HYP
    cycle_start: "2026-07-01"
```

The `cycle_start` dates above are placeholders. The project owner will supply the correct dates. Do not hardcode them anywhere in the Python logic — they must always come from YAML.

---

## Implementation Approach

### Step 1 — Read and map the existing code
Before writing a single line, run the existing tool and confirm the current output matches `Master_Program_2026_2028_v1.xlsx`. If there is a `Makefile`, `setup.py`, or `pyproject.toml`, read it to understand the entry point.

### Step 2 — Implement `cycle_engine.py`
Create a new module (do not modify existing modules yet) with a single public function:

```python
def get_phase_for_date(
    target_date: date,
    cycle_start: date,
    holidays: set[date]
) -> str:
    """
    Given a calendar date, the stream's cycle start date, and a set of
    non-working dates (public holidays + school holidays + personal leave),
    return the phase code for that date.

    Returns one of: 'FRE', 'VER', 'VAL', 'REL', 'PCO', 'BRK'
    Returns None if the date is before cycle_start.
    """
```

The function must count only working days when measuring phase duration,
but return a phase code for every calendar date (including weekends and holidays)
based on where in the cycle that date falls.

### Step 3 — Wire into existing calendar generator
Replace the logic that writes `R`, `S`, `G` to `PROG` cells with calls to `cycle_engine.get_phase_for_date()`. Do not restructure the existing Excel writing code — make the minimum change needed.

### Step 4 — Apply colors
Replace the old color mappings (`FFFADADD` for R, etc.) with the new color table above.

### Step 5 — Update YAML schema
Add `cycle_start` to the stream configuration schema and update the YAML reader accordingly.

---

## Testing Requirements

After implementation, verify:

1. Run the tool and open the generated `.xlsx`
2. Confirm `FRE` appears for exactly 1 working day at the start of each stream's first cycle
3. Confirm `VER` spans exactly 10 working days after `FRE`
4. Confirm `VAL` spans exactly 69 working days after `VER`
5. Confirm `REL` spans exactly 10 working days after `VAL`
6. Confirm `PCO` spans exactly 10 working days after `REL`
7. Confirm `BRK` spans exactly 5 working days after `PCO`
8. Confirm the cycle then repeats with a new `FRE`
9. Confirm that Swiss public holidays, school holidays, and personal leave days do NOT advance the phase counter (i.e. the phase on the day after a holiday is the same phase as before it, and the budget is not consumed)
10. Confirm all `HOL`, `Dat`, `WD` rows are unchanged from the baseline output
11. Confirm colors match the table above exactly (check hex via openpyxl cell inspection)

---

## Hard Rules — Do Not Violate

- **Never modify `HOL` row logic.** The holiday and leave computation is correct and must not be touched.
- **Never hardcode dates.** All stream cycle start dates come from YAML.
- **Never change the Excel column structure.** 69 columns, 2 months side by side, separator at column 33/34.
- **Never change the band row structure** beyond replacing `R`/`S`/`G` with the new codes.
- **Weekends and holidays still get phase codes** — they are not blank in `PROG`. Only the duration counter skips them.
- **Do not add new rows** to the calendar bands. The row structure is fixed.
- **Version the output file** — the existing tool auto-increments the version number (v1, v2, v3…). Preserve this behaviour.

---

## Reference: Current Phase Colors Being Replaced

For your records — the old colors that are being superseded:

| Old Code | Old Color | Replaced By |
|---|---|---|
| `R` (Ready) | `FFFADADD` | `FRE` + `VER` + `VAL` |
| `S` (Steady) | `FFFFF9C4` | `VAL` continues / `REL` |
| `G` (Go/No-Go) | `FFC8E6C9` | `REL` |

---

## Questions to Raise Before Proceeding

If any of the following are unclear after reading the source code, stop and ask the project owner:

1. What is the exact entry point command to run the tool?
2. Where does the existing phase-to-date assignment logic live? (Which function / file?)
3. Are there multiple PROG rows (one per stream) or a single PROG row? The current Excel shows one `PROG` row per band — clarify whether multiple streams should each have their own row, or whether a single stream is shown at a time.
4. What are the correct `cycle_start` dates for each of the 4 streams (FAB, FLA, ORC, HYP)?
