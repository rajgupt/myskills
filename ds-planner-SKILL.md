---
name: ds-planner
description: Use when starting any data science task that involves multiple
  stages of work where downstream choices depend on upstream findings. Triggers
  include: "build a model", "estimate the effect of X on Y", "analyze this
  dataset", "forecast", "detect anomalies", or any multi-step ML/stats/causal
  workflow. Produces a role-based phased plan in plan.md with explicit
  checkpoints and a state machine for each phase. Do NOT use for single-shot
  queries like "plot this column" or "fit a logistic regression on this data".
---

# Data Science Planner

## Operating principle

Plans are **provisional past the next checkpoint**. Detail the immediate phase;
sketch later phases as hypotheses tagged with the assumptions they depend on.
Replan after every checkpoint by producing a **diff against the previous plan**,
not a rewrite.

## Step 1: Classify the work

Before drafting phases, classify the work type. This determines which phase
**roles** are needed. Common types:

| Type | Typical roles needed |
|---|---|
| Predictive ML (classification/regression) | frame → audit → construct → estimate → stress-test → communicate |
| Causal inference | frame → audit (incl. overlap/identification) → construct → estimate → stress-test (sensitivity) → communicate |
| Forecasting | frame → audit → construct (decomposition) → estimate → stress-test (backtest) → communicate |
| Exploratory / hypothesis generation | frame → audit → construct (lightweight) → stress-test (confirmatory checks) → communicate |
| Anomaly detection | frame → audit (define normality) → construct → estimate → stress-test (drift) → communicate |
| NLP / representation learning | frame → audit (corpus) → construct (preprocessing/embedding) → estimate (task model) → stress-test → communicate |

If ambiguous, **ask the user** before drafting. Bad classification is the
single biggest cause of wasted phases downstream.

## Step 2: Phase roles (the stable abstraction)

Every DS workflow phase plays one of these roles. Names vary; roles don't.

| Role | What it does | Checkpoint question |
|---|---|---|
| **frame** | Lock goal, metric, constraints, scope | Is the question answerable with this data? |
| **audit** | Understand what you have; check assumptions | Did expected properties hold? |
| **construct** | Build inputs downstream needs (features, instruments, embeddings) | Are constructed inputs valid and non-leaky? |
| **estimate** | Produce the primary output (model, effect, forecast, clusters) | Is the result statistically credible? |
| **stress-test** | Try to break what you built | Does it survive adversarial probing? |
| **communicate** | Hand off (report, dashboard, deployment) | Will stakeholders correctly interpret it? |

Audit and stress-test roles can repeat. Some projects skip estimate entirely
(pure EDA). Pick 4–6 roles per project.

## Step 3: Instantiate phases

Map roles to project-specific phase names. Example for a causal project:

```
P0 (frame):       Identification strategy & estimand definition
P1 (audit):       Data quality + temporal structure
P2 (audit):       Overlap & positivity check
P3 (construct):   Treatment/outcome variables + covariate set
P4 (estimate):    DML with cross-fitting
P5 (stress-test): Sensitivity to unobserved confounding
P6 (communicate): Effect estimate + CI for stakeholders
```

## Plan structure (plan.md)

Every plan contains:

1. **Goal** — question, success metric, constraints, out-of-scope items
2. **Work classification** — type + one-line justification
3. **Phase status table** — id, role, name, state (see state machine below)
4. **Active phase** — full detail: steps, expected outputs, decision criteria
5. **Provisional phases** — bullets + explicit dependencies on upstream results
6. **Open questions** — only items the human can decide
7. **Decision log** — append-only record of replans, with diffs

## Phase state machine

Each phase moves through:

```
PLANNED → IN_EXECUTION → AWAITING_REVIEW → (RE)PLANNED ──┐
                                              │           │
                                              └─ DONE ────┘
```

- `PLANNED`: defined in plan.md, not started
- `IN_EXECUTION`: ds-phase-executor working on it
- `AWAITING_REVIEW`: phase complete, waiting for ds-checkpoint-reviewer + human
- `(RE)PLANNED`: human approved revision; downstream phases updated
- `DONE`: human approved as-is; downstream phases unchanged

The agent **never** transitions a phase to DONE or (RE)PLANNED without
explicit human input. No silent state changes.

## Checkpoint trigger criteria

Always checkpoint at phase boundaries. Additionally, **interrupt mid-phase** if:

- Target leakage suspected
- Distribution shift or class imbalance discovered that invalidates the metric
- Methodology assumption violated (e.g., DML overlap fails, stationarity fails for forecasting)
- Baseline performance far from expected (either direction — surprise is information)
- Key feature/variable has unexpected semantics or >X% missingness
- The original success metric no longer makes sense given what's learnable

## Replan as diff, not rewrite

When findings invalidate downstream phases, the checkpoint reviewer produces
a **diff**: which phases change, what changes, why. This:

- Keeps the audit trail clean
- Anchors the agent to history (prevents drift)
- Makes the human's review job tractable (review changes, not the whole plan)

Format diffs as:

```
~ P3 (estimate): was "logistic regression baseline"
                 now "logistic regression baseline, stratified by cohort"
                 reason: P1 found 2x churn rate gap between cohorts

+ P4.5 (audit): added "promo cohort error analysis"
                reason: subgroup performance gap discovered in P3

- P5 step 3: removed "calibration on full data"
             reason: superseded by per-cohort calibration in revised P5
```

## When to invoke other skills

- After plan.md is drafted: hand off to **ds-phase-executor** for the active phase.
- When ds-phase-executor reports phase complete: invoke **ds-checkpoint-reviewer**.
- Never execute the next phase without a completed checkpoint review.

## Anti-patterns

- Drafting all phases in full detail upfront (creates false confidence)
- Skipping work classification ("it's ML" — but is it predictive or causal?)
- Rewriting plan.md instead of producing diffs
- Auto-advancing past AWAITING_REVIEW
