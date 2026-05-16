---
name: S2-eda-investigation
description: STUB. a validated spec and data profile exist and hypotheses need to be investigated and patterns exposed for feature planning. Make sure to use this skill whenever the agentic data science pipeline reaches the EDA Investigation stage, or when the user mentions exploratory data analysis, EDA, testing hypotheses, finding patterns, leakage detection, or feature ideas.
---

# S2 — EDA Investigation

> **Status: STUB.** Infrastructure (orchestration, contract validation,
> ledgering, loop control) is handled by `../../PLUGIN.md`. This skill only
> needs the four sections below filled in.

## Role in the pipeline

The hypothesis-driven search stage. Confirms/rejects spec hypotheses, surfaces feature candidates, flags leakage and data limitations, and feeds open questions back to S0.

## 1. Pre-contract (consumes)

**Input contract type(s):** `SpecContract` + `DataProfileContract`

> Fill in: the exact `payload` fields this skill reads, and which are
> hard pre-conditions (missing them ⇒ emit `BLOCKED`) vs. soft
> (missing them ⇒ proceed with a recorded `assumption`).

| Field | From | Hard? | Used for |
|-------|------|-------|----------|
| _TBD_ | _TBD_ | _TBD_ | _TBD_ |

## 2. Post-contract (produces)

**Output contract type:** `EDAFindingsContract`
**Payload schema:** `../../contracts/EDAFindingsContract.schema.json`

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
| target mis-defined / missing segment / wrong data assumption discovered | S2->S0 | S0 | revised target def or added segment in spec | |
