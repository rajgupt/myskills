Phase Roles (P0 → P5)

Each phase is assigned a role — a stable abstraction that defines what the phase does and what
question it must answer before proceeding.

┌───────┬─────────────┬────────────────────────────────────────────┬───────────────────────┐
│ Phase │    Role     │                What it does                │  Checkpoint question  │
├───────┼─────────────┼────────────────────────────────────────────┼───────────────────────┤
│       │             │ Lock the goal, success metric,             │ Is the question       │
│ P0    │ frame       │ constraints, and scope. Define what "done" │ answerable with this  │
│       │             │  looks like before any data is touched.    │ data?                 │
├───────┼─────────────┼────────────────────────────────────────────┼───────────────────────┤
│       │             │ Understand what you actually have. Check   │ Did expected          │
│ P1    │ audit       │ data quality, distributions, structure,    │ properties hold?      │
│       │             │ and assumptions.                           │                       │
├───────┼─────────────┼────────────────────────────────────────────┼───────────────────────┤
│       │             │ Build the inputs downstream phases need —  │ Are constructed       │
│ P2    │ construct   │ features, instruments, embeddings,         │ inputs valid and      │
│       │             │ treatment/outcome variables.               │ non-leaky?            │
├───────┼─────────────┼────────────────────────────────────────────┼───────────────────────┤
│       │             │ Produce the primary output — a model,      │ Is the result         │
│ P3    │ estimate    │ causal effect estimate, forecast, or       │ statistically         │
│       │             │ clusters.                                  │ credible?             │
├───────┼─────────────┼────────────────────────────────────────────┼───────────────────────┤
│       │             │ Actively try to break what was built.      │ Does it survive       │
│ P4    │ stress-test │ Adversarial probing, sensitivity analysis, │ adversarial probing?  │
│       │             │  backtests, drift checks.                  │                       │
├───────┼─────────────┼────────────────────────────────────────────┼───────────────────────┤
│       │             │ Hand off the result — report, dashboard,   │ Will stakeholders     │
│ P5    │ communicate │ or deployment.                             │ correctly interpret   │
│       │             │                                            │ it?                   │
└───────┴─────────────┴────────────────────────────────────────────┴───────────────────────┘
