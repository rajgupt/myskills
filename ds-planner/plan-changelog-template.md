# Plan Changelog: <project name>

> Append-only. Never edit past entries. Each entry added after human approves
> a checkpoint. Committed together with plan.md.

---

### YYYY-MM-DD — Initial plan
- Work classified as <type>.
- Phases P0–P<n> drafted with roles: <list>.
- Commit: `plan: initial draft`

---

### YYYY-MM-DD — Checkpoint after P<id>
**Blast radius:** local | cascade | goal
**Findings summary:**
- <bullet>
- <bullet>
- <bullet>

**Changes applied:**
```
~ P<id> (<role>): <old> → <new>
                  reason: <evidence>
+ P<id> (<role>): added "<name>"
                  reason: <evidence>
- P<id> step <n>: removed "<step>"
                  reason: <evidence>
```

**Human decision:** <option chosen + any flagged concerns>
**Commit:** `checkpoint: P<id> <done|replan> — <one-line reason>`
