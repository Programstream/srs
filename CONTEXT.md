# Programstream Context

## Program Tool Reference

This document provides shared context for both the **SRS Agent** and **TSD Agent** to understand the existing Program tool that will be converted into a web application.

### What is Program?

**Program** is a Python-based **Master Program Calendar Generator** for project management (2026-2028 timeframe).

**Repository**: https://github.com/Programstream/program  
**Type**: Command-line tool + Excel output  
**Purpose**: Generate multi-project timelines using RSG (Ready-Steady-Go) framework

### Key Capabilities

1. **Constraint-Aware Planning**
   - Integrates public holidays (Swiss calendar, Canton Zurich)
   - School holidays (ISCS Cham)
   - Personal leave allocations
   - Team capacity buffers

2. **RSG Framework Implementation**
   - **READY** (Prepare): Document collection and gap-filling
   - **STEADY** (Validate): Stakeholder validation and risk assessment
   - **GO/NO-GO** (Decide): Final decision with evidence
   - Each phase has variable duration based on project complexity

3. **Multi-Stream Management**
   - Tracks multiple projects simultaneously
   - Shows phase allocations across timeline
   - Manages overlapping cycles
   - Holistic program-level planning

4. **Output**
   - Excel workbooks with calendar layout
   - Visual representation of phases per month
   - Constraint layer (holidays, personal leave)
   - Version control with auto-incrementing versions

### Data Structure

**Input Files (YAML)**
- `holidays.yaml` — Public and school holiday calendar
- `master_program.yaml` — Program structure, phases, capacity config
- Project configs per stream (FAB, FLA, ORC, HYP)

**Output**
- `Master_Program_2026_2028_vN.xlsx` — Excel calendar with timeline

### Technology Stack (Current)

- **Language**: Python 3.x
- **Key Libraries**: openpyxl (Excel generation), pyyaml (config parsing)
- **Data Format**: YAML input, Excel output
- **CLI**: Entry point via setuptools

### Key Concepts for Web Conversion

1. **Calendar Grid Layout**
   - 31-column structure per month
   - 4 rows per month band (Date, Weekday, Holidays, Programme)
   - Extensible to support real-time updates

2. **Color-Coded Constraints**
   - Weekend: Light gray
   - Public holidays: Red
   - School holidays: Blue
   - Personal leave: Orange

3. **Phase Visualization**
   - READY (R): Pastel red
   - STEADY (S): Pastel yellow
   - GO/NO-GO (G): Pastel green

4. **Versioning Pattern**
   - Auto-incrementing versions: v1, v2, v3...
   - Allows users to track evolution of the program

---

## Collaboration Instructions

**For SRS Agent:**
- Focus on: What functionality must the web tool provide to match Program capabilities?
- Validate: Are all current features translatable to web UI?
- Ask TSD: Is this requirement technically feasible?

**For TSD Agent:**
- Focus on: How to implement SRS requirements with modern web architecture?
- Validate: Can we improve upon the Excel output with interactive features?
- Ask SRS: Do web enhancements conflict with the original requirements?

---

## References

- Program GitHub: https://github.com/Programstream/program
- RSG Framework Docs: https://github.com/Programstream/program/blob/main/RSG-PROGRAM.md
- Calendar Generator Code: https://github.com/Programstream/program/blob/main/Road%20Map%2026-27/src/roadmap_calendar/calendar_gen.py
