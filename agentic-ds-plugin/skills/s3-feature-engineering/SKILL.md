---
name: S3-feature-engineering
description: STUB. EDA findings exist and leakage-safe features with documented lineage need to be built. Make sure to use this skill whenever the agentic data science pipeline reaches the Feature Engineering stage, or when the user mentions feature engineering, feature transformations, building model inputs, or feature lineage.
---

# S3 — Feature Engineering

> **Status: STUB.** Infrastructure (orchestration, contract validation,
> ledgering, loop control) is handled by `../../PLUGIN.md`. This skill only
> needs the four sections below filled in.

## Role in the pipeline

Turns EDA-recommended candidates into concrete, leakage-safe, train/serve-consistent features.

## 1. Pre-contract (consumes)

**Input contract type(s):** `EDAFindingsContract`

> Fill in: the exact `payload` fields this skill reads, and which are
> hard pre-conditions (missing them ⇒ emit `BLOCKED`) vs. soft
> (missing them ⇒ proceed with a recorded `assumption`).

| Field | From | Hard? | Used for |
|-------|------|-------|----------|
| _TBD_ | _TBD_ | _TBD_ | _TBD_ |

## 2. Post-contract (produces)

**Output contract type:** `FeatureSetContract`
**Payload schema:** `../../contracts/FeatureSetContract.schema.json`

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
| a needed relationship was never probed by EDA | S3->S2 | S2 | targeted re-investigation of that relationship | |
