# Programme: <Programme Name>

> Shared context for all projects under this programme.
> Project plans reference this file — do not duplicate contents in plan.md.
> Update this file when shared facts change; note the date and reason inline.

---

## Programme Objective

**Business goal:** <what outcome does this programme serve — one paragraph>

**Stakeholders:**
| Role | Name/Team | What they care about |
|------|-----------|----------------------|
| Sponsor | | |
| Primary consumer | | |
| Data owner | | |

**Timeline / milestones:** <key dates, review cycles, deployment windows>

---

## Shared Vocabulary

> Misaligned definitions are the most common cross-project failure. Lock these here.

| Term | Definition | Notes / edge cases |
|------|------------|--------------------|
| User | | |
| Event | | |
| Conversion | | |
| <domain term> | | |

---

## Datasets

### <Dataset Name>
- **Location:** <path / system / access method>
- **Owner:** <team or person>
- **Grain:** <one row = one what?>
- **Date range:** <coverage>
- **Known issues:** <missingness, drift, label lag, encoding quirks>
- **Do not use for:** <known misuse patterns>

### <Dataset Name>
- ...

---

## Shared Constraints

**Data access:** <what requires approval, what's restricted, PII rules>

**Compute:** <infra available, cost limits, job scheduling rules>

**Modelling:** <interpretability requirements, banned techniques, regulatory constraints>

**Deployment:** <target system, latency/throughput requirements, monitoring expectations>

**Reporting cadence:** <when findings are expected, to whom>

---

## Cross-Project Findings

> Append-only. Date every entry. These are facts discovered in one project that
> affect others — surprises, invalidated assumptions, shared gotchas.

- **YYYY-MM-DD** [Project: X] <finding and implication for other projects>
- **YYYY-MM-DD** [Project: Y] <finding and implication>

---

## Decisions Made at Programme Level

> Append-only. These are settled — projects inherit them, not re-litigate them.

- **YYYY-MM-DD** <decision> — *Reason: <why>*
- **YYYY-MM-DD** <decision> — *Reason: <why>*

---

## Project Registry

| Project | Question | State | plan.md |
|---------|----------|-------|---------|
| <name> | <one sentence> | active / complete / paused | <path> |
| <name> | | | |

---

## Project Dependencies

> "data" = one project's output feeds another's input.
> "assumption" = one project's finding validates/invalidates another's approach.
> Update when dependencies are discovered, not just at programme start.

| Upstream project | Downstream project | Type | What is shared / assumed | Risk if upstream changes |
|------------------|--------------------|------|--------------------------|--------------------------|
| <Project A> | <Project B> | data | <e.g., scored user segments> | <e.g., B's model retrains on new segments> |
| <Project C> | <Project B> | assumption | <e.g., treatment randomness holds> | <e.g., B must switch to IV approach if violated> |

---

## What Each Project plan.md Should NOT Repeat

- Shared vocabulary (reference this file)
- Dataset descriptions (reference this file)
- Programme-level constraints (reference this file)

Each plan.md should open with:
> "Operates under `<path>/programme.md` — shared definitions, datasets,
> and constraints apply unless explicitly overridden below."

Each plan.md Goal section should include:
```
**Upstream dependencies:** <project name> — <what we consume and when it's expected>
**Downstream dependents:** <project name> — <what we produce and what they need from it>
```
