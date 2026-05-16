# Governance Adapter

Binds the `ds-*` governance skills (in `../governance/`) to the contract
backbone (S0–S6). The two operate on **orthogonal axes** and compose
through this adapter — no rewrite of either side.

- **Contract axis** (`PLUGIN.md`): *what* the work is. Typed contracts
  between stages S0–S6, validated by the orchestrator.
- **Governance axis** (`ds-*` skills): *how* the work is governed. A
  plan → execute → human-checkpoint loop with append-only audit.

| Governance skill | Plays the role of | Defined in |
|---|---|---|
| `ds-planner` | The orchestrator (`PLUGIN.md §6`) — instantiates phases, owns the plan, replans as diff | `governance/ds-planner/` |
| `ds-phase-executor` | The skill runner — runs exactly one S-skill, emits its contract | `governance/ds-phase-executor/` |
| `ds-checkpoint-reviewer` | The contract validator **+** human gate — validates `post(N) ⊨ pre(N+1)`, routes feedback edges | `governance/ds-checkpoint-reviewer/` |

---

## Role ↔ skill ↔ contract map

The planner's six **roles** cover the seven **S-skills**. `audit` expands
to two skills because profiling and EDA are distinct contracts.

| ds role      | S-skill                   | Output contract        |
|--------------|---------------------------|------------------------|
| frame        | S0 Problem Framing        | `SpecContract`         |
| audit        | S1 Data Profiling         | `DataProfileContract`  |
| audit        | S2 EDA Investigation      | `EDAFindingsContract`  |
| construct    | S3 Feature Engineering    | `FeatureSetContract`   |
| estimate     | S4 Modeling               | `ModelContract`        |
| stress-test  | S5 Evaluation             | `EvaluationContract`   |
| communicate  | S6 Reporting              | `DeliverableContract`  |

When `ds-planner` instantiates phases (its Step 3), each phase is tagged
with the S-skill it runs and the contract it must emit. The phase status
table in `plan.md` gains two columns: `s_skill` and `contract_type`.

---

## Binding 1 — Status ↔ phase-state

The contract envelope `status` and the planner's phase state machine are
kept in sync by `ds-checkpoint-reviewer` at every checkpoint.

| Envelope `status` | Phase state after executor | After human review |
|-------------------|----------------------------|--------------------|
| `COMPLETE`        | `AWAITING_REVIEW`          | `DONE`             |
| `NEEDS_REVISION`  | `AWAITING_REVIEW`          | `(RE)PLANNED`      |
| `PARTIAL`         | `AWAITING_REVIEW`          | reviewer decides: `DONE` if downstream pre-contract tolerates it, else `(RE)PLANNED` |
| `BLOCKED`         | `AWAITING_REVIEW` + escalate flag | human must resolve `violations[]` before any transition |

**Key rule:** the executor **never** auto-advances. A skill emitting any
status lands in `AWAITING_REVIEW`. This is how the human gate (governance
axis) wraps the orchestrator's auto-validation (contract axis) — see
"Cycle control" below.

---

## Binding 2 — Findings block ↔ contract payload

`ds-phase-executor` already writes a role-specific **Findings block** to
`plan.md`. The adapter adds one serialization step at the end of the
executor's rule 5:

1. Write the human-readable Findings block (unchanged — executor's existing behavior).
2. **Also** serialize the same content into a contract: wrap it in the
   envelope (`contracts/contract_envelope.schema.json`) with `payload`
   validated against `contracts/<ContractType>.schema.json`.
3. Append the contract to `ledger/run_<id>/` (the Ledger *is* the
   machine-readable twin of `plan-changelog.md`).

The Findings block is the human view; the contract is the machine view.
Same information, two renderings. Role → payload correspondence:

| Executor Findings role | Populates payload of |
|------------------------|----------------------|
| frame      | `SpecContract`        |
| audit      | `DataProfileContract` (S1) or `EDAFindingsContract` (S2) |
| construct  | `FeatureSetContract`  |
| estimate   | `ModelContract`       |
| stress-test| `EvaluationContract`  |
| communicate| `DeliverableContract` |

---

## Binding 3 — Blast radius ↔ feedback edge

When `ds-checkpoint-reviewer` produces a diff, its **blast radius** tag is
resolved to a named feedback edge and written into the envelope's
`revision_request` block (which carries the loop-termination evidence).

| Reviewer blast radius | Named edge (PLUGIN.md §4) | Envelope status |
|-----------------------|---------------------------|-----------------|
| `local`               | none — in-skill re-run, no edge | `NEEDS_REVISION`, `revision_request.edge = null`, same S-skill re-entered |
| `cascade`             | `S3→S2` or `S4→S3` (reviewer picks by which phases the diff touches) | `NEEDS_REVISION` + matching edge |
| `goal`                | `S2→S0` or `S5→S0` (reframing) | `NEEDS_REVISION` + matching edge |

The reviewer fills `revision_request.reason` from the Findings evidence and
`revision_request.changed` from the proposed diff's right-hand side — this
is exactly the data the orchestrator's iteration-budget check needs.

---

## Cycle control: human gate outside, budget inside

The one genuine philosophy mismatch — automatic (budget) vs. human
(approval) loop termination — is resolved by **layering**, not choosing:

1. A skill finishes → executor lands it in `AWAITING_REVIEW` (never
   auto-routes, even on `NEEDS_REVISION`).
2. `ds-checkpoint-reviewer` produces a Checkpoint Report with the proposed
   edge (Binding 3) and the A/B questions. **It waits for the human.**
3. Human approves the loop → reviewer transitions phase to `(RE)PLANNED`
   and routes along the named edge.
4. **Only now** does the orchestrator's per-edge iteration budget
   (`orchestrator/config.yaml`) apply: if this edge has already been
   traversed ≥ budget times, the orchestrator overrides to `BLOCKED` and
   escalates — catching the case where a human keeps approving an
   S2↔S0 ping-pong.

So: **human approval is the outer control loop; the iteration budget is an
inner safety backstop.** Neither side's skill logic changes.

---

## End-to-end flow (one phase)

```
ds-planner
  └─ instantiate phase  (tag: s_skill=S2, contract_type=EDAFindingsContract)
     plan.md: phase PLANNED

ds-phase-executor  (handles exactly this one phase)
  ├─ phase → IN_EXECUTION
  ├─ run S2's Procedure (skills/s2-eda/SKILL.md)
  ├─ write Findings block (audit role) → plan.md
  ├─ serialize → EDAFindingsContract envelope → ledger/      [Binding 2]
  └─ phase → AWAITING_REVIEW                                  [Binding 1]

ds-checkpoint-reviewer  (always invoked, never skipped)
  ├─ validate payload vs EDAFindingsContract.schema.json
  ├─ validate post(S2) ⊨ pre(S3)        (contract-axis check)
  ├─ audit downstream provisional phases
  ├─ if defect: blast radius → named edge → revision_request  [Binding 3]
  ├─ produce Checkpoint Report + A/B questions
  └─ WAIT for human
        ├─ approve as-is  → phase DONE,  advance to S3
        ├─ approve diff   → phase (RE)PLANNED, route edge
        │                    └─ orchestrator budget check      [Cycle control]
        └─ reject/redirect→ stays AWAITING_REVIEW

ds-planner
  └─ replan as diff (plan.md current truth; plan-changelog.md append-only)
```

---

## Files the governance axis owns

These live in the project workspace, alongside `ledger/`:

| File | Owner skill | Role on contract axis |
|------|-------------|------------------------|
| `plan.md` | ds-planner / reviewer | Human-readable view of phase status + active contract bindings |
| `plan-changelog.md` | reviewer | Human-readable twin of the Ledger; records every replan + edge taken |
| `ledger/run_<id>/*.json` | executor (via Binding 2) | Machine-checkable contract history |

`plan.md` = current truth. `plan-changelog.md` + `ledger/` = full history.
