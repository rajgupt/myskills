# Memory File Schemas

Genie parses these files on every session. Format drift breaks the skill. Follow these schemas exactly.

## INDEX.md

One screen, scanned first every session. Pure markdown, no tables.

```markdown
# Genie Memory — Index

_Last updated: YYYY-MM-DD_

## Tool files
- `tools/databricks_sdk.md` — N avoid, M verified
- `tools/databricks_api_v2.md` — N avoid, M verified
- `tools/pyspark.md` — N avoid, M verified
- `tools/unity_catalog.md` — N avoid, M verified
- `tools/mlflow.md` — N avoid, M verified

## Top recurring failures (last 30 days)
1. `<call signature>` — N occurrences — see `tools/<file>.md`
2. ...

## Recent additions
- YYYY-MM-DD: Added `<call>` to avoid in `databricks_sdk.md`
- ...
```

## tools/<tool_name>.md

Three sections, in this order, always present (even if empty):

```markdown
# <tool_name>

_Last updated: YYYY-MM-DD_

## ❌ Avoid

### `<exact call signature>`
- **Reason**: <one-line root cause>
- **Error**: `<exception type>: <short message>`
- **First seen**: YYYY-MM-DD
- **Failure count**: N
- **Alternative**: <working call signature, or "ask user" if none known>
- **Notes**: <optional, only if non-obvious context needed>

### `<next entry>`
...

## ✅ Verified Working

### `<exact call signature>`
- **Confirmed**: YYYY-MM-DD
- **Context**: <when this works, e.g. "DBR 14.3+, Unity Catalog enabled">
- **Notes**: <optional>

## ⚠️ Conditional

### `<call signature>`
- **Works when**: <precondition>
- **Fails when**: <condition>
- **First seen**: YYYY-MM-DD
```

**Rules:**
- Call signatures use Python syntax with type hints where the type matters: `w.jobs.run_now(job_id: int, notebook_params: dict[str, str])`.
- Never include actual job IDs, cluster IDs, user names, or secrets in signatures or notes. Use `<job_id>`, `<cluster_id>`, etc.
- Reason field is one line. If the explanation needs more, link to a section in `patterns/anti_patterns.md`.

## session_log.md

Append-only. New entries at the **top** (reverse chronological).

```markdown
# Session Log

| Date | Tool | Call | Error | Promoted? |
|------|------|------|-------|-----------|
| 2026-05-09 | databricks_sdk | `w.jobs.run_now(job_id, notebook_params={"a": {"b": 1}})` | `BadRequest: notebook_params must be flat` | yes → databricks_sdk.md |
| 2026-05-09 | pyspark | `df.writeTo("cat.sch.tbl").append()` | `AnalysisException: table does not exist` | no (transient — table created later) |
```

**Rules:**
- One row per failure. Never edit existing rows.
- "Promoted?" is one of: `yes → <file>`, `no (<reason>)`, `pending`.
- If a row stays `pending` for 7+ days, surface it to the user next session.

## patterns/working_recipes.md

End-to-end snippets that have been confirmed working. Each recipe is self-contained.

```markdown
# Working Recipes

## <Recipe name>

_Confirmed: YYYY-MM-DD_

**Use when:** <one-line trigger condition>

**Code:**
```python
# minimal working example
```

**Why this works:** <one-line explanation, only if non-obvious>

---
```

## patterns/anti_patterns.md

Architectural mistakes that span multiple tools or sessions.

```markdown
# Anti-Patterns

## <Anti-pattern name>

_First identified: YYYY-MM-DD_

**Pattern:** <description of what Genie was trying to do>

**Why it fails:** <root cause>

**Do this instead:** <correct approach, with link to recipe if applicable>

---
```

## What never goes in any file

- Secrets, tokens, passwords, API keys (even in error messages — redact)
- Full stack traces (one-line error message only)
- Query results or actual data
- Personal user info beyond the workspace user_name in the path
- Workspace URLs or instance names
