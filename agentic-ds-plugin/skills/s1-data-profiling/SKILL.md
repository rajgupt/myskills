---
name: S1-data-profiling
description: STUB. the pipeline needs schema, grain, join integrity, volume and sentinel checks before any analysis. Make sure to use this skill whenever the agentic data science pipeline reaches the Data Profiling stage, or when the user mentions data profiling, schema checks, data sanity, grain or join validation.
---

# S1 — Data Profiling

> **Status: STUB.** Infrastructure (orchestration, contract validation,
> ledgering, loop control) is handled by `../../PLUGIN.md`. This skill only
> needs the four sections below filled in.

## Role in the pipeline

Establishes that the data is structurally what the spec assumed before any modeling-oriented analysis begins.

## 1. Pre-contract (consumes)

**Input contract type(s):** `SpecContract`

> Fill in: the exact `payload` fields this skill reads, and which are
> hard pre-conditions (missing them ⇒ emit `BLOCKED`) vs. soft
> (missing them ⇒ proceed with a recorded `assumption`).

| Field | From | Hard? | Used for |
|-------|------|-------|----------|
| _TBD_ | _TBD_ | _TBD_ | _TBD_ |

## 2. Post-contract (produces)

**Output contract type:** `DataProfileContract`
**Payload schema:** `../../contracts/DataProfileContract.schema.json`

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
| spec assumes a grain/join that the data violates | (validation fail, not a named edge) | S0 via orchestrator repair_route | corrected data inventory / grain in spec | |
