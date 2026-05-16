---
name: S0-problem-framing
description: STUB. the user is starting a new data science problem and needs the objective, target, hypotheses, data inventory and guardrails specified. Make sure to use this skill whenever the agentic data science pipeline reaches the Problem Framing stage, or when the user mentions problem framing, modeling objectives, target definition, success metrics, or hypotheses to test.
---

# S0 — Problem Framing

> **Status: STUB.** Infrastructure (orchestration, contract validation,
> ledgering, loop control) is handled by `../../PLUGIN.md`. This skill only
> needs the four sections below filled in.

## Role in the pipeline

Entry point of the pipeline. Translates a business objective into a machine-checkable spec that every downstream skill is bound by. Also the re-entry point for the S2→S0 and S5→S0 feedback edges.

## 1. Pre-contract (consumes)

**Input contract type(s):** (bootstrap) business objective + data access

> Fill in: the exact `payload` fields this skill reads, and which are
> hard pre-conditions (missing them ⇒ emit `BLOCKED`) vs. soft
> (missing them ⇒ proceed with a recorded `assumption`).

| Field | From | Hard? | Used for |
|-------|------|-------|----------|
| _TBD_ | _TBD_ | _TBD_ | _TBD_ |

## 2. Post-contract (produces)

**Output contract type:** `SpecContract`
**Payload schema:** `../../contracts/SpecContract.schema.json`

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
| _no upstream — receives S2→S0 / S5→S0 revisions instead_ | — | — | — | |
