# Agentic Data Science Plugin

A contract-validated, cyclical skill set covering the full data science
process — from problem framing through delivery. Each skill is a swappable
black box bounded by a **pre-contract** (what it needs to start) and a
**post-contract** (what it guarantees on exit). Contracts are the
checkpoints: they make the pipeline auditable, resumable, and safely
iterative.

---

## 1. Mental model

The process is a **state machine over contracts**, not a linear script.

- A **skill** transforms an input contract into an output contract.
- The **orchestrator** sits between skills. Its only job at each checkpoint
  is to validate that `post_contract(N)` satisfies `pre_contract(N+1)`.
  If it does not, the orchestrator does not advance — it routes to a
  feedback edge.
- A shared **Ledger** (see §5) persists every contract emitted, so the run
  can resume, branch, or roll back to any checkpoint.

Because skills only talk to each other through contracts, you can plan,
rewrite, or replace any single skill later without touching the rest of the
pipeline — as long as the new skill honors the same contract shapes.

---

## 2. The skill backbone

| #  | Skill                         | Consumes (pre)                              | Produces (post)            |
|----|-------------------------------|---------------------------------------------|----------------------------|
| S0 | Problem Framing / Spec Author | (bootstrap) business objective + data access | `SpecContract`             |
| S1 | Data Profiling / Sanity       | `SpecContract`                              | `DataProfileContract`      |
| S2 | EDA / Hypothesis Investigation| `SpecContract` + `DataProfileContract`      | `EDAFindingsContract`      |
| S3 | Feature Engineering           | `EDAFindingsContract`                       | `FeatureSetContract`       |
| S4 | Modeling / Experimentation    | `FeatureSetContract` + `SpecContract`       | `ModelContract`            |
| S5 | Evaluation / Validation       | `ModelContract` + `SpecContract`            | `EvaluationContract`       |
| S6 | Reporting / Handoff           | `EvaluationContract` + all prior contracts  | `DeliverableContract`      |

Each skill lives in `skills/<id>/SKILL.md` with four sections to be filled
in later: **Pre-contract**, **Post-contract**, **Procedure**,
**Feedback triggers**. The contract envelope and the per-skill payload
schemas live in `contracts/`.

---

## 2a. The governance layer (how skills get run)

§2 is the **contract axis** — *what* the work is. The S0–S6 skills are
black boxes that transform contracts. They do not run themselves, gate on
humans, or replan. That is the **governance axis**, supplied by three
skills vendored in `governance/`:

| Governance skill | Responsibility | Maps to |
|---|---|---|
| `ds-planner` | Instantiates phases, owns `plan.md`, replans as a diff (never a rewrite) | The orchestration loop in §6 |
| `ds-phase-executor` | Runs exactly **one** S-skill, writes its Findings + emits its contract | "`run(skill, pre)`" in §6 |
| `ds-checkpoint-reviewer` | Validates `post(N) ⊨ pre(N+1)`, resolves feedback edges, **gates on the human** | "`validate(...)`" + `repair_route` in §6 |

The two axes are orthogonal and compose through a thin adapter — three
table-sized bindings, no rewrite of either side. The adapter is specified
in **`orchestrator/adapter.md`** (read it before implementing the loop):

1. **Status ↔ phase-state** — envelope `status` ⇆ planner state machine
2. **Findings ↔ payload** — executor's Findings block is serialized into
   the contract envelope and appended to the Ledger
3. **Blast radius ↔ feedback edge** — reviewer's `local|cascade|goal` tag
   resolves to a named edge (§4) + `revision_request`

The planner's six **roles** cover the seven S-skills (audit → S1 *and* S2):
frame→S0, audit→S1/S2, construct→S3, estimate→S4, stress-test→S5,
communicate→S6.

**Cycle control resolves the one philosophy clash.** Governance terminates
loops by *human approval* at `AWAITING_REVIEW`; the contract axis
terminates them by *iteration budget* (§4). These layer rather than
conflict: the human gate is the **outer** control; the per-edge budget is
an **inner** backstop that fires only if a human keeps approving the same
loop past its budget. Full mechanics in `orchestrator/adapter.md` →
"Cycle control".

---

## 3. The contract envelope

Every contract — regardless of skill — shares one envelope so the
orchestrator can validate generically. The skill-specific part lives only
in `payload`. The canonical schema is in
`contracts/contract_envelope.schema.json`; the per-skill payloads are in
`contracts/<contract_name>.schema.json` (stubs for now).

```
Contract
├── id                 # unique id for this contract instance
├── skill_name         # which skill emitted it (S0..S6)
├── contract_type      # e.g. "SpecContract"
├── version            # schema version of the payload
├── timestamp          # emission time (UTC ISO-8601)
├── status             # COMPLETE | PARTIAL | BLOCKED | NEEDS_REVISION
├── iteration          # which cycle of this skill produced it (1-indexed)
├── payload            # { ...skill-specific schema... }
├── assumptions[]      # things taken as given, surfaced for review
├── violations[]       # pre-conditions that were expected but not met
├── open_questions[]   # ambiguities that feed the feedback loop
└── provenance         # { inputs_used[], artifacts[], code_ref, env_hash }
```

### Status semantics

| Status           | Meaning                                                  | Orchestrator action                              |
|------------------|----------------------------------------------------------|--------------------------------------------------|
| `COMPLETE`       | Post-contract fully satisfied                            | Validate against next skill's pre-contract, advance |
| `PARTIAL`        | Usable but incomplete (e.g. sampled, time-boxed)         | Advance only if next skill's pre-contract tolerates it |
| `BLOCKED`        | Cannot proceed; a hard pre-condition was unmet           | Halt, surface `violations[]` to the human        |
| `NEEDS_REVISION` | Skill discovered an upstream defect                      | Route along the matching feedback edge (§4)      |

---

## 4. Cycles: feedback edges

Iteration is **explicit**, never accidental. Each feedback edge has a named
trigger and a target skill. A skill emits `status = NEEDS_REVISION` plus a
`revision_request` block in its payload identifying the edge.

| Edge      | Trigger (why it fires)                                                 | Routes to |
|-----------|------------------------------------------------------------------------|-----------|
| S2 → S0   | EDA reveals mis-defined target, missing segment, wrong data assumption | S0        |
| S3 → S2   | Feature engineering needs a relationship EDA never probed              | S2        |
| S4 → S3   | Model error analysis points to a missing or weak feature               | S3        |
| S5 → S4   | Metric miss attributable to model choice, not data                     | S4        |
| S5 → S0   | Objective is infeasible with available data; reframe needed            | S0        |

### Loop termination

Every feedback edge traversal must record, in the contract:

- `iteration` — incremented per skill re-entry
- `revision_request.reason` — why the loop was taken
- `revision_request.changed` — what is expected to differ next pass

The orchestrator enforces a per-edge **iteration budget** (default 3,
configurable in `orchestrator/config.yaml`). Exceeding it forces a `BLOCKED`
status and escalates to the human, which is how an infinite S2↔S0 ping-pong
is detected and capped.

---

## 5. The Ledger

`orchestrator/` maintains an append-only Ledger of every contract emitted:

```
ledger/
├── run_<id>/
│   ├── S0_iter1_SpecContract.json
│   ├── S1_iter1_DataProfileContract.json
│   ├── S2_iter1_EDAFindingsContract.json   # status: NEEDS_REVISION -> S0
│   ├── S0_iter2_SpecContract.json          # revised spec
│   ├── S2_iter2_EDAFindingsContract.json
│   └── ...
```

This gives free resumability (restart from the last `COMPLETE` checkpoint),
branching (fork a run at any contract), and a complete audit trail of why
every cycle was taken.

---

## 6. The orchestration loop (governance-driven)

The loop is **operated by the three `ds-*` skills**, not by standalone
code. Below, each step names the governance skill that performs it; the
adapter bindings (`orchestrator/adapter.md`) are cited as `[B1]`/`[B2]`/`[B3]`.

```
# ds-planner: classify work, instantiate phases, tag each with
# (s_skill, contract_type). plan.md created; phases PLANNED.
plan = ds_planner.draft()

while plan has a PLANNED phase:
    phase = lowest_id(PLANNED)              # ds-planner ordering

    # ---- ds-phase-executor: exactly one S-skill ----
    pre = gather_inputs(phase.s_skill.pre_contract, ledger)
    if not validate(pre, pre_contract_schema(phase.s_skill)):
        phase -> AWAITING_REVIEW (+escalate)            # [B1]
    else:
        phase -> IN_EXECUTION
        out = run(phase.s_skill, pre)      # skills/<id>/SKILL.md Procedure
        write_findings_block(out)          # human view -> plan.md
        ledger.append(serialize_contract(out))          # [B2]
        phase -> AWAITING_REVIEW                         # [B1] never auto-advance

    # ---- ds-checkpoint-reviewer: validate + human gate ----
    validate(out.payload, <ContractType>.schema.json)
    contract_ok = validate(out, pre_contract_schema(successor))
    if defect: revision_request = blast_radius_to_edge(diff)  # [B3]
    report = build_checkpoint_report(out, contract_ok, A/B questions)
    human = WAIT_FOR_HUMAN(report)         # hard gate — outer control loop

    match human:
        approve_as_is:                     # status COMPLETE
            phase -> DONE
            advance plan to successor S-skill
        approve_diff:                      # status NEEDS_REVISION
            phase -> (RE)PLANNED
            edge = revision_request.edge
            if edge and NOT within_budget(edge):         # inner backstop
                emit BLOCKED ; escalate ; break
            ds_planner.replan_as_diff(edge)              # plan.md + changelog
        reject_redirect:
            phase stays AWAITING_REVIEW ; ds_planner.adjust()

deliver when S6 phase reaches DONE
```

The contract-axis checks (`validate post ⊨ pre`) live inside
`ds-checkpoint-reviewer`; the loop-termination backstop
(`within_budget`) is the only place the orchestrator config overrides a
human decision, and only to escalate. See `orchestrator/adapter.md` →
"Cycle control" for why the human gate and the budget compose rather than
conflict.

---

## 7. What you fill in per skill, later

For each `skills/<id>/SKILL.md` you only need to specify four things —
orchestration, validation, ledgering, and loop control are all handled by
this framework:

1. **Pre-contract** — the exact payload fields the skill consumes
2. **Post-contract** — the exact payload fields the skill produces
3. **Procedure** — the layered internal steps the skill runs
4. **Feedback triggers** — conditions under which it emits `NEEDS_REVISION`,
   and which edge it routes to

Everything else in this document is fixed infrastructure.

---

## 8. Directory map

```
agentic-ds-plugin/
├── PLUGIN.md                       # this file
├── contracts/
│   ├── contract_envelope.schema.json
│   ├── SpecContract.schema.json
│   ├── DataProfileContract.schema.json
│   ├── EDAFindingsContract.schema.json
│   ├── FeatureSetContract.schema.json
│   ├── ModelContract.schema.json
│   ├── EvaluationContract.schema.json
│   └── DeliverableContract.schema.json
├── governance/                     # the "how": vendored ds-* skills (§2a)
│   ├── ds-planner/                 # orchestration loop owner
│   ├── ds-phase-executor/          # runs one S-skill, emits its contract
│   └── ds-checkpoint-reviewer/     # validator + human gate
├── orchestrator/
│   ├── README.md
│   ├── adapter.md                  # governance ↔ contract bindings (B1/B2/B3)
│   └── config.yaml                 # iteration budgets, sampling defaults
└── skills/                         # the "what": S0–S6 contract stages
    ├── s0-problem-framing/SKILL.md
    ├── s1-data-profiling/SKILL.md
    ├── s2-eda/SKILL.md
    ├── s3-feature-engineering/SKILL.md
    ├── s4-modeling/SKILL.md
    ├── s5-evaluation/SKILL.md
    └── s6-reporting/SKILL.md

# Per-project workspace (created at run time, not in the plugin):
#   plan.md  plan-changelog.md  ledger/run_<id>/*.json
```
