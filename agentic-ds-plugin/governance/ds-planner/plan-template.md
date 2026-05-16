# Project Plan: <project name>

> Living document. Reflects current state only — no history inline.
> Full checkpoint history (findings, diffs, decisions) lives in `plan-changelog.md`.
> Both files committed together after every approved checkpoint.

---

## Goal

**Question:** <one sentence>

**Success metric:** <concrete, measurable, with target>

**Constraints:** <data, compute, deadline, interpretability, regulatory>

**Out of scope:** <explicit non-goals>

---

## Work classification

**Type:** <predictive ML / causal inference / forecasting / exploratory / anomaly detection / NLP / ...>

**Justification:** <one line — why this type, not another>

**Roles needed:** <list of 4–6 roles from: frame, audit, construct, estimate, stress-test, communicate>

---

## Phase Status

| Phase | Role         | Name                                     | State            |
|-------|--------------|------------------------------------------|------------------|
| P0    | frame        | <project-specific name>                  | DONE             |
| P1    | audit        | <project-specific name>                  | IN_EXECUTION     |
| P2    | construct    | <project-specific name>                  | PLANNED          |
| P3    | estimate     | <project-specific name>                  | PLANNED          |
| P4    | stress-test  | <project-specific name>                  | PLANNED          |
| P5    | communicate  | <project-specific name>                  | PLANNED          |

States: `PLANNED → IN_EXECUTION → AWAITING_REVIEW → (RE)PLANNED or DONE`

---

## Active phase: P<id> (<role>) — <name>

### Steps
1. <concrete step with expected output>
2. ...

### Expected outputs
- <artifact / finding>

### Decision criteria
- **Proceed if:** <conditions>
- **Replan if:** <conditions specific to this role's checkpoint question>

---

## Provisional phases (sketched)

### P<id> (<role>): <name>
- Plan: <brief>
- **Depends on:** <which upstream findings must hold for this plan to remain valid>

### P<id> (<role>): <name>
- ...

---

## Open questions

- [ ] <A/B/C question requiring human input>

---

## Decision Log

See `plan-changelog.md` for full history of checkpoint findings, diffs, and
human decisions. This file reflects current state only.
