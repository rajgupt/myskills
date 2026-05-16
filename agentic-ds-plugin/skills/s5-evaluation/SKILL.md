---
name: S5-evaluation
description: STUB. a trained model exists and must be judged against the success metric, fairness and robustness criteria in the spec. Make sure to use this skill whenever the agentic data science pipeline reaches the Evaluation stage, or when the user mentions evaluation, validation, model acceptance, go/no-go, fairness or robustness checks.
---

# S5 — Evaluation

> **Status: STUB.** Infrastructure (orchestration, contract validation,
> ledgering, loop control) is handled by `../../PLUGIN.md`. This skill only
> needs the four sections below filled in.

## Role in the pipeline

Judges the model against the spec's definition-of-done and decides go/no-go, routing back to S4 (model issue) or S0 (infeasible objective).

## 1. Pre-contract (consumes)

**Input contract type(s):** `ModelContract` + `SpecContract`

> Fill in: the exact `payload` fields this skill reads, and which are
> hard pre-conditions (missing them ⇒ emit `BLOCKED`) vs. soft
> (missing them ⇒ proceed with a recorded `assumption`).

| Field | From | Hard? | Used for |
|-------|------|-------|----------|
| _TBD_ | _TBD_ | _TBD_ | _TBD_ |

## 2. Post-contract (produces)

**Output contract type:** `EvaluationContract`
**Payload schema:** `../../contracts/EvaluationContract.schema.json`

> Fill in: the exact `payload` fields this skill guarantees on
> `COMPLETE`, and what a `PARTIAL` looks like (which fields may be
> omitted/sampled and still be downstream-usable).

| Field | Type | Required on COMPLETE | Notes |
|-------|------|----------------------|-------|
| _TBD_ | _TBD_ | _TBD_ | _TBD_ |

## 3. Procedure

> Fill in: the layered internal steps. Keep them ordered and idempotent so
> a re-entry after a feedback loop is safe.

1. _TBD_

## 4. Feedback triggers

> Fill in: conditions under which this skill emits
> `status = NEEDS_REVISION` plus a `revision_request`, and which edge it
> takes. Edges and their iteration budgets are defined in
> `../../PLUGIN.md` §4 and `../../orchestrator/config.yaml`.

| Condition | Edge | Routes to | What must change next pass |
|-----------|------|-----------|-----------------------------|
| metric miss due to model choice | S5->S4 | S4 | re-experiment with different model/params |\n| objective infeasible with available data | S5->S0 | S0 | reframed objective or relaxed definition-of-done | |
