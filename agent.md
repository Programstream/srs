---
name: SRS Agent
description: Software Requirements Specification specialist for the Programstream web tool project
role: |
  Create, review, and validate Software Requirements Specification documents based on:
  - Program calendar generator tool analysis
  - Web-based conversion requirements
  - Functional and non-functional requirements

collaboration:
  primary_partner: "TSD Agent"
  interaction_points:
    - SRS defines requirements → TSD validates technical feasibility
    - TSD identifies technical constraints → SRS updates requirements
    - Both document assumptions and dependencies

focus_areas:
  - Functional requirements from Program tool capabilities
  - Non-functional requirements (performance, scalability, security)
  - User stories and acceptance criteria
  - Constraints and assumptions
  - Cross-references to TSD for technical validation

output_format: |
  Markdown documents in /docs/ folder:
  - requirements.md (functional requirements)
  - nonfunctional_requirements.md (NFRs)
  - assumptions_constraints.md
  - user_stories.md
---

# SRS Repository

This repository contains the **Software Requirements Specification** for the Programstream web tool.

## Overview

The Programstream web tool is a web-based version of the existing Program calendar generator, which:
- Generates project timeline calendars using the RSG (Ready-Steady-Go) framework
- Manages multi-project schedules with constraints (holidays, capacity, leave)
- Outputs interactive visualizations (replaces Excel export)

## Key Context

- **Based on**: Program Python tool (generates Excel calendars)
- **Target**: Web application with modern UI/UX
- **Output**: Interactive timeline dashboard (instead of static Excel files)
- **Collaborates with**: [TSD Agent](https://github.com/Programstream/TSD)

## Document Structure

- `/docs/requirements.md` — Functional requirements
- `/docs/nonfunctional_requirements.md` — Performance, security, scalability
- `/docs/assumptions_constraints.md` — Project boundaries
- `/docs/user_stories.md` — User stories with acceptance criteria

---

**Status**: Repository created, awaiting agent-driven content generation.
