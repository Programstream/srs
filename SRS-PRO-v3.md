# SRS-PRO — Software Requirements Specification
## Program — Master Calendar Generator
### Version 3

---

## What Changed in v3

Two precise changes only. Everything else from v2 remains identical.

### Change 1 — PRE `BRK` renamed to `HDV` (Hand Over)

The break week at the end of the PRE stream is **not** a neutral rest period. It is the **handover moment** — the point at which the build is complete and the programme pivots to the POST delivery cycle. Naming it `BRK` was misleading and visually lost against weekend gray. It is now `HDV`.

**What this means in the code:**
- Find every place `BRK` is assigned to a PRE stream cell
- Replace with `HDV`
- Update the YAML config accordingly (see updated YAML below)
- Do NOT rename POST `BRK` — that remains `BRK` and is unchanged

### Change 2 — `HDV` and POST `BRK` share warm amber color. Gray removed from both streams.

`BRK` (gray `FFE0E0E0`) was a problem — it is visually identical to weekend cells in the `WD` row which are also gray. This made transition weeks disappear into the weekend noise.

Both transition/rest moments — PRE `HDV` and POST `BRK` — now share **warm amber `FFFFD54F`**. Same color, same meaning: *pause, transition, breathe*.

**What this means in the code:**
- Remove `FFE0E0E0` gray from both streams entirely
- Apply `FFFFD54F` to all `HDV` cells in PRE
- Apply `FFFFD54F` to all `BRK` cells in POST
- `FFE0E0E0` gray is now **only used by the `WD` row for weekends** — nowhere else

---

## Full Updated Phase Tables

### PRE stream — complete (replaces v2 PRE table):

| Code | Full Name | Duration | Cycle weeks | Color name | Hex |
|---|---|---|---|---|---|
| `DEF` | Define | 2 calendar weeks | Weeks 1–2 | Soft lavender | `FFE8EAF6` |
| `SCP` | Scope | 2 calendar weeks | Weeks 3–4 | Medium lavender | `FFC5CAE9` |
| `BLD` | Build | 3 calendar weeks | Weeks 5–7 | Deep lavender | `FF9FA8DA` |
| `HDV` | Hand Over | 1 calendar week | Week 8 | Warm amber | `FFFFD54F` |

**PRE total: 8 weeks active + 1 week HDV = 9 weeks**

### POST stream — complete (replaces v2 POST table):

| Code | Full Name | Duration | Cycle weeks | Color name | Hex |
|---|---|---|---|---|---|
| `FRE` | Freeze | 1 calendar day | Week 9 day 1 | Deep slate blue | `FF5C6BC0` |
| `VER` | Verify | 2 calendar weeks | Weeks 9–10 | Pastel teal | `FFB2DFDB` |
| `VAL` | Validate | 3 calendar weeks | Weeks 11–13 | Pastel yellow | `FFFFF9C4` |
| `REL` | Release | 2 calendar weeks | Weeks 14–15 | Pastel green | `FFC8E6C9` |
| `PCO` | Post Cut Over | 1 calendar week | Week 16 | Pastel peach | `FFFFE0B2` |
| `BRK` | Break | 1 calendar week | Week 17 | Warm amber | `FFFFD54F` |

**POST total: ~8 weeks active + 1 week BRK = ~9 weeks**

---

## Visual Alignment — HDV and FRE Column

`HDV` (PRE week 8) and `FRE` (POST week 9 day 1) sit in adjacent rows on the **same Monday column**. This is intentional. On that Monday the calendar shows:

```
PRE row:  ... BLD BLD | HDV HDV HDV HDV HDV | DEF ...
POST row: ...         | FRE | VER VER VER ...
```

- `HDV` amber spans the full week in PRE
- `FRE` slate blue sits on the Monday of that same week in POST — 1 cell wide
- Together they form a **visible pivot column** — the eye reads it as the handover moment even without a legend

No special rendering logic is needed for this alignment — it emerges naturally from the phase timing. Do not add any extra logic to force alignment.

---

## Updated Color Palette — Complete Reference

This is the authoritative color table for the entire calendar. No other colors should appear in PRE or POST rows.

| Stream | Code | Full Name | Hex |
|---|---|---|---|
| PRE | `DEF` | Define | `FFE8EAF6` |
| PRE | `SCP` | Scope | `FFC5CAE9` |
| PRE | `BLD` | Build | `FF9FA8DA` |
| PRE | `HDV` | Hand Over | `FFFFD54F` |
| POST | `FRE` | Freeze | `FF5C6BC0` |
| POST | `VER` | Verify | `FFB2DFDB` |
| POST | `VAL` | Validate | `FFFFF9C4` |
| POST | `REL` | Release | `FFC8E6C9` |
| POST | `PCO` | Post Cut Over | `FFFFE0B2` |
| POST | `BRK` | Break | `FFFFD54F` |

**Color rules:**
- `FFE0E0E0` gray → **weekends only** (`WD` row). Never in PRE or POST.
- `FFFFD54F` amber → **`HDV` in PRE and `BRK` in POST only**. Same color, same meaning in both streams.
- `FF5C6BC0` slate blue → **`FRE` only**. Single-day marker. Must remain visually dominant.

---

## Updated YAML Configuration

Replace the PRE section in the YAML. The only change from v2 is `BRK` → `HDV`:

```yaml
cycle:
  pre:
    - code: DEF
      weeks: 2
    - code: SCP
      weeks: 2
    - code: BLD
      weeks: 3
    - code: HDV       # ← renamed from BRK in v2
      weeks: 1
  post:
    - code: FRE
      days: 1
    - code: VER
      weeks: 2
    - code: VAL
      weeks: 3
    - code: REL
      weeks: 2
    - code: PCO
      weeks: 1
    - code: BRK       # ← unchanged
      weeks: 1
```

---

## Updated Testing Checklist (changes from v2 only)

Replace v2 test items 3, 4, and 20 with the following:

**Item 3 (was: PRE BLD)** — unchanged, still 3 calendar weeks ✓

**Item 4 (was: PRE BRK):**
> PRE `HDV` follows `BLD` immediately for exactly 1 calendar week. Cells display code `HDV` and color `FFFFD54F`. No cell in the PRE row displays `BRK` anywhere in the 2026–2028 output.

**Item 20 (was: both BRK gray):**
> `HDV` cells in PRE display `FFFFD54F` amber. `BRK` cells in POST display `FFFFD54F` amber. No cell in PRE or POST rows displays `FFE0E0E0` gray anywhere in the output. Gray `FFE0E0E0` appears only in the `WD` row on Saturdays and Sundays.

**New item 23:**
> On the Monday that `FRE` fires in POST, the PRE row shows `HDV` on that same column. Visually confirm the amber + slate blue pivot column is visible in the printed calendar.

---

## Hard Rules Added in v3

In addition to all hard rules from v2:

- **`HDV` is the correct code for PRE week 8.** Never write `BRK` to a PRE row cell.
- **`BRK` is the correct code for POST week 17.** Never write `HDV` to a POST row cell.
- **Gray `FFE0E0E0` must not appear in PRE or POST rows.** If you find yourself applying it to a phase cell, stop — use `FFFFD54F` for `HDV` and `BRK`.

---

## No Other Changes

Everything else in v2 remains authoritative:
- Excel band structure and row offsets — unchanged
- Cycle timing and week counts — unchanged
- YAML programme start date and validation — unchanged
- Implementation steps 1–6 — unchanged
- All other testing items — unchanged
- All other hard rules — unchanged

Refer to SRS-PRO-v2 for all sections not covered in this document.
