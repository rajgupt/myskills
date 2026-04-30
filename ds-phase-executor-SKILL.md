---
name: ds-phase-executor
description: Execute a single phase of a data science plan. Use after
  ds-planner has produced plan.md and a phase is in PLANNED state. Transitions
  the phase to IN_EXECUTION, runs the work, then transitions to AWAITING_REVIEW
  and hands off to ds-checkpoint-reviewer. Handles exactly one phase per
  invocation. Do NOT use to execute multiple phases sequentially — that
  bypasses the human-in-loop gate between phases.
---

# DS Phase Executor

## Rules

1. Read `plan.md` first. Identify the phase in `PLANNED` state with the lowest id.
2. Transition that phase to `IN_EXECUTION` in plan.md before starting work.
3. Execute only that phase. If you find yourself wanting to "just quickly do
   the next thing too" — stop. That's the gate working.
4. Write code + commentary as you go. Surface anomalies aggressively; do not
   smooth over weird results.
5. End the phase by writing a **Findings block** to plan.md, transitioning the
   phase to `AWAITING_REVIEW`, and invoking ds-checkpoint-reviewer.

## Findings block format

The format depends on the phase's role. Match the role's checkpoint question.

### For `frame` phases
```
## P<id> Findings (frame)
- Goal locked as: <restatement>
- Success metric: <metric, with rationale>
- Key constraints: <list>
- Identified risks to feasibility: <list>
- Recommendation: proceed / refine goal / escalate
```

### For `audit` phases
```
## P<id> Findings (audit)
- Properties checked: <list>
- Assumptions that held: <list with evidence>
- Assumptions that broke: <list with evidence and severity>
- Surprises (unexpected patterns, even if not strictly assumption violations): <list>
- Implications for downstream phases: <which provisional phases now need revision>
- Recommendation: proceed / replan from P<id> / escalate
```

### For `construct` phases
```
## P<id> Findings (construct)
- Inputs built: <list with brief description>
- Validity checks performed: <list — leakage, missingness, distribution sanity>
- Inputs that failed validity: <list with reason>
- Univariate signal (where applicable): <summary>
- Implications for estimate phase: <which model assumptions are now supported / threatened>
- Recommendation: proceed / construct more / escalate
```

### For `estimate` phases
```
## P<id> Findings (estimate)
- Method used: <with rationale tied to constructed inputs>
- Primary result: <point estimate + uncertainty>
- Performance vs. success metric: <gap analysis>
- Diagnostic checks: <residuals, calibration, balance, etc. as appropriate>
- Subgroup performance: <where applicable>
- Implications for stress-test phase: <what to probe hardest>
- Recommendation: proceed / re-estimate / replan upstream / escalate
```

### For `stress-test` phases
```
## P<id> Findings (stress-test)
- Probes performed: <list>
- Probes survived: <list>
- Probes that revealed weakness: <list with severity>
- Robustness verdict: <strong / moderate / weak / not deployment-ready>
- Implications for communication phase: <what caveats must be conveyed>
- Recommendation: proceed / re-estimate / replan / escalate
```

### For `communicate` phases
```
## P<id> Findings (communicate)
- Artifacts produced: <list>
- Audience considerations: <how interpretation risk was managed>
- Open items handed back to stakeholder: <list>
- Recommendation: proceed to handoff / revise based on feedback
```

## Anti-patterns

- Silently fixing data issues without flagging them in Findings
- Tuning hyperparameters before establishing a baseline (skips evidence)
- Reporting only point estimates without uncertainty
- Continuing into the next phase after Findings — that's the reviewer's job
- Generic Findings text that doesn't match the phase's role
- Treating Findings as a status report rather than evidence for the checkpoint
