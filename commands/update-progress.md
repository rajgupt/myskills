# /update-progress

Guide a conversation to capture manual work done on a ds-planner phase and update plan.md in-place, exactly as ds-phase-executor would.

## Rules

- Read plan.md and plan-changelog.md first. Understand all phases, their states, and the full context.
- One phase per conversation. Never update multiple phases in one run.
- Ask all questions before writing anything to plan.md.
- Mirror ds-phase-executor's exact output format — Findings block, state transitions, changelog entries.
- Deviations from the plan go to plan-changelog.md only, never inline in plan.md.
- Never invent or assume details the user didn't provide. Ask if unclear.

## Step 1: Read the plan

Read `plan.md` and `plan-changelog.md` from the current working directory. If plan.md doesn't exist, stop and tell the user.

Identify:
- All phases and their current states
- The active phase (IN_EXECUTION or most recently started)
- The phase role (frame / audit / construct / estimate / stress-test / communicate)
- The steps and expected outputs defined for that phase
- Any assumptions or dependencies declared

## Step 2: Confirm the phase

Tell the user which phases exist and their states. Ask:

> "Which phase did you work on?"

Show the phase list with names and states so the user can pick. Once confirmed, show the user what was planned for that phase (steps, expected outputs, decision criteria) so they have context for the questions ahead.

## Step 3: Ask the guided questions

Ask these questions one at a time, in order. Wait for the answer before asking the next. Do not batch them.

**Q1 — Completion status:**
> "Did you complete this phase fully, or is it still in progress?"

If in progress: the phase will be updated to IN_EXECUTION with partial findings.
If complete: the phase will be updated to AWAITING_REVIEW with full Findings block.

**Q2 — What you did:**
> "Walk me through what you did, step by step. You can be informal — I'll structure it."

**Q3 — Findings:**
Ask the role-specific findings questions (see below). Ask each sub-question individually.

**Q4 — Deviations:**
> "Did you do anything differently from what was planned? For example, different method, different data, skipped a step, or added something not in the plan?"

If yes: ask for the specific deviation, the reason, and the blast radius (did it affect only this phase, or does it change downstream phases?).

**Q5 — Blockers / open questions:**
> "Any blockers you hit, or open questions that need a decision before the next phase?"

**Q6 — Next steps:**
> "What do you think should happen next — proceed to the next phase, replan something, or escalate?"

## Role-specific findings questions (Q3)

Match the phase role and ask only the relevant sub-questions.

### frame
- "What did you lock as the goal — the exact question being answered?"
- "What success metric did you define, and why?"
- "What are the key constraints or out-of-scope items?"
- "Any risks to feasibility you identified?"
- "Your recommendation: proceed, refine the goal, or escalate?"

### audit
- "What properties or assumptions did you check?"
- "Which assumptions held? What's the evidence?"
- "Which assumptions broke? What's the evidence, and how severe?"
- "Any surprises — unexpected patterns, even if not a strict assumption violation?"
- "Which downstream phases does this affect, and how?"
- "Your recommendation: proceed, replan from which phase, or escalate?"

### construct
- "What inputs did you build? Brief description of each."
- "What validity checks did you run — leakage, missingness, distribution sanity?"
- "Any inputs that failed validity? Why?"
- "Any univariate signal worth noting?"
- "What does this imply for the estimate phase — which model assumptions are supported or threatened?"
- "Your recommendation: proceed, construct more, or escalate?"

### estimate
- "What method did you use, and why given the constructed inputs?"
- "What's the primary result — point estimate and uncertainty?"
- "How does it compare to the success metric? What's the gap?"
- "What diagnostic checks did you run — residuals, calibration, balance, etc.?"
- "Any subgroup performance differences worth noting?"
- "What does this imply for the stress-test phase — what should be probed hardest?"
- "Your recommendation: proceed, re-estimate, replan upstream, or escalate?"

### stress-test
- "What probes did you run?"
- "Which probes survived?"
- "Which revealed weakness? How severe?"
- "Robustness verdict: strong, moderate, weak, or not deployment-ready?"
- "What caveats must the communicate phase convey?"
- "Your recommendation: proceed, re-estimate, replan, or escalate?"

### communicate
- "What artifacts did you produce?"
- "Any audience considerations — how did you manage interpretation risk?"
- "Any open items handed back to the stakeholder?"
- "Your recommendation: proceed to handoff, or revise based on feedback?"

## Step 4: Confirm before writing

Summarize what you're about to write:
- The new phase state
- The Findings block content
- Any changelog entry for deviations

Ask: "Does this look right before I update plan.md?"

Only proceed after explicit confirmation.

## Step 5: Write the updates

### Update plan.md in-place

1. Update the phase status table — change the phase state to `IN_EXECUTION` (partial) or `AWAITING_REVIEW` (complete).
2. Under the phase's section in plan.md, append the Findings block using the exact format from ds-phase-executor (see formats below).
3. Do not rewrite any other part of plan.md.

### Findings block formats

Use the format that matches the phase role, exactly as ds-phase-executor would write it.

#### frame
```
## P<id> Findings (frame)
- Goal locked as: <restatement>
- Success metric: <metric, with rationale>
- Key constraints: <list>
- Identified risks to feasibility: <list>
- Recommendation: proceed / refine goal / escalate
```

#### audit
```
## P<id> Findings (audit)
- Properties checked: <list>
- Assumptions that held: <list with evidence>
- Assumptions that broke: <list with evidence and severity>
- Surprises (unexpected patterns, even if not strictly assumption violations): <list>
- Implications for downstream phases: <which provisional phases now need revision>
- Recommendation: proceed / replan from P<id> / escalate
```

#### construct
```
## P<id> Findings (construct)
- Inputs built: <list with brief description>
- Validity checks performed: <list — leakage, missingness, distribution sanity>
- Inputs that failed validity: <list with reason>
- Univariate signal (where applicable): <summary>
- Implications for estimate phase: <which model assumptions are now supported / threatened>
- Recommendation: proceed / construct more / escalate
```

#### estimate
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

#### stress-test
```
## P<id> Findings (stress-test)
- Probes performed: <list>
- Probes survived: <list>
- Probes that revealed weakness: <list with severity>
- Robustness verdict: <strong / moderate / weak / not deployment-ready>
- Implications for communication phase: <what caveats must be conveyed>
- Recommendation: proceed / re-estimate / replan / escalate
```

#### communicate
```
## P<id> Findings (communicate)
- Artifacts produced: <list>
- Audience considerations: <how interpretation risk was managed>
- Open items handed back to stakeholder: <list>
- Recommendation: proceed to handoff / revise based on feedback
```

### Update plan-changelog.md (only if there are deviations)

Append a new entry in this format:

```markdown
### YYYY-MM-DD — Manual update after P<id> (<role>)
**Blast radius:** local | cascade | goal
**Findings summary:**
- <bullet>
- <bullet>

**Changes applied:**
~ P<id> (<role>): <old description> → <new description>
                 reason: <evidence from findings>
+ P<id> (<role>): added "<new phase name>"
                  reason: <evidence>
- P<id> step <n>: removed "<step>"
                  reason: <evidence>

**Human decision:** <what the user chose + any flagged concerns>
```

If there are no deviations, do not append to plan-changelog.md.

## Step 6: Handoff

After writing:
- If phase is `AWAITING_REVIEW`: tell the user the phase is ready for checkpoint review and suggest invoking ds-checkpoint-reviewer.
- If phase is `IN_EXECUTION`: summarize what's captured and what remains to complete the phase.

## Anti-patterns

- Asking all questions at once instead of one at a time
- Writing to plan.md before confirming with the user
- Inventing findings the user didn't provide
- Writing history or deviations inline in plan.md (changelog only)
- Updating more than one phase per run
- Using a different Findings format than what ds-phase-executor uses
