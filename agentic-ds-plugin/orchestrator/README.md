# Orchestrator

This directory holds the **glue** between the two axes of the plugin. It
contains no skill logic of its own — the logic lives in `../governance/`
(the ds-* skills) and `../skills/` (S0–S6).

## Files

| File | Purpose |
|------|---------|
| `adapter.md` | The integration spec. Defines the three bindings (status↔state, findings↔payload, blast-radius↔edge) and the cycle-control layering. **Read this first.** |
| `config.yaml` | Per-edge iteration budgets and run defaults. The inner loop-termination backstop. |

## How a run works (one paragraph)

`ds-planner` reads the user's objective, classifies the work, and writes
`plan.md` with phases tagged by S-skill and contract type. `ds-phase-executor`
runs exactly one phase — invoking that phase's `skills/<id>/SKILL.md`
Procedure — then writes both a human-readable Findings block (to `plan.md`)
and a machine-checkable contract (to `ledger/`). `ds-checkpoint-reviewer`
validates the contract against its schema and against the next skill's
pre-contract, resolves any defect to a named feedback edge, and **waits for
the human**. On approval it transitions phase state and, if a loop was
approved, the orchestrator checks `config.yaml` budgets as a backstop
before `ds-planner` replans as a diff.

## To make this runnable later

The plugin currently specifies the system; it does not execute it. To
operationalize, implement a thin driver that:

1. invokes the three governance skills in the order in `PLUGIN.md §6`,
2. enforces `config.yaml` budgets at the one point `adapter.md` specifies,
3. validates contracts with any JSON-Schema validator against
   `../contracts/*.schema.json`.

Everything that driver needs is already specified in `adapter.md` — it is
intentionally small (the heavy lifting is in the skills).
