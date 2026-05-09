 ---
name: genie-self-improvement
description: Persistent failure-memory skill for Databricks Genie Code. Use this skill at the START of every Genie session and after EVERY tool/SDK failure. It maintains a long-term memory of which databricks-sdk calls, REST API v2.0 endpoints, PySpark patterns, and Python package usages have failed before, so Genie stops repeating the same mistakes across sessions. Trigger this skill whenever the user asks Genie to do anything involving the Databricks SDK, Databricks REST API, Unity Catalog, jobs, clusters, warehouses, notebooks, DBFS/volumes, MLflow, or any Python tool call inside Genie. Also trigger when a tool call fails — failures must be captured, not silently retried.
---

# Genie Self-Improvement

A skill for Databricks Genie Code that turns failed tool calls into persistent, indexed knowledge. Genie does not have CLI/npm access — it operates purely through Python and the Databricks SDK / REST API v2.0 — so its failure surface is narrow and repetitive. This skill exploits that: every confirmed failure is written to a markdown file Genie reads on the next session.

## Core principle

**Read before you call. Log when you fail. Ask before you forbid.**

Three rules, in order of priority:

1. Before any non-trivial tool call, check the memory for known failures and verified-working patterns.
2. After any failure, capture it to the session log immediately.
3. Only promote a failure to the permanent "avoid" list after the user confirms it's a real (not transient) issue.

## Memory location

All memory lives at:

```
/dbfs/Users/<current_user>/.genie_memory/
```

DBFS path is canonical because Genie runs across workspaces and projects — a single source of truth beats per-project drift. If `/dbfs/` is unavailable in the current runtime (e.g. Unity Catalog-only workspaces), fall back to `/Volumes/main/default/genie_memory/` and tell the user.

Resolve `<current_user>` via:

```python
from databricks.sdk import WorkspaceClient
w = WorkspaceClient()
user = w.current_user.me().user_name
```

## Memory layout

```
.genie_memory/
├── INDEX.md                    # 1-screen summary, read first every session
├── tools/
│   ├── databricks_sdk.md       # Failures + working patterns, per tool
│   ├── databricks_api_v2.md
│   ├── pyspark.md
│   ├── unity_catalog.md
│   ├── mlflow.md
│   └── _template.md            # Copy this when adding a new tool file
├── patterns/
│   ├── working_recipes.md      # Verified end-to-end snippets
│   └── anti_patterns.md        # Confirmed-bad architectural approaches
└── session_log.md              # Append-only raw failure log (markdown table)
```

## Workflow

### Step 1 — Session start (mandatory)

Before doing anything the user asked for, read in this order:

1. `INDEX.md` — gives the lay of the land
2. The `tools/*.md` files relevant to the current task (e.g. if the user asks to run a job, read `databricks_sdk.md` and `databricks_api_v2.md`)
3. `patterns/anti_patterns.md` if the task is architectural (designing a pipeline, picking between APIs, etc.)

Do this silently. Do not narrate "I'm reading my memory now." Just absorb it and proceed.

If `.genie_memory/` does not exist yet, create it from the templates in `references/templates/` (see "Bootstrap" section below) and tell the user once: "Initializing Genie memory at `<path>`. Future sessions will use this automatically."

### Step 2 — Pre-call check

Before any Databricks SDK, REST API, or non-trivial Python call:

- Match the planned call against entries in the relevant `tools/*.md`.
- If it matches a `❌ Avoid` entry: **do not run it**. Use the documented alternative, or if no alternative exists, ask the user how to proceed.
- If it matches a `✅ Verified Working` entry: use the exact signature given.
- If it matches a `⚠️ Conditional` entry: confirm the precondition holds before calling.
- If it matches nothing: proceed normally, but be ready to capture failure.

### Step 3 — Failure capture

When a tool call raises an exception or returns an error response:

1. **Immediately** append a row to `session_log.md` (see schema in `references/schemas.md`). This is non-negotiable and happens before anything else.
2. Inspect the error. Classify it informally:
   - **Likely transient**: network timeouts, 5xx responses, rate limits, auth-token-expired. Note in log, do not promote.
   - **Likely real**: `TypeError`, `ValueError`, schema mismatches, "method does not exist", 4xx with clear message, silent wrong-output. These are candidates for promotion.
3. Ask the user **once**, concisely:

   > "This call failed: `<call signature>` with error `<short error>`. This looks like a [real / transient] failure. Should I add it to the permanent avoid-list for `<tool>`? (yes / no / it's transient)"

4. Based on the user's answer:
   - **yes** → add to the relevant `tools/*.md` under `❌ Avoid` with reason, alternative (if known), and date. Set failure count to 1, or increment if entry already exists.
   - **no / transient** → leave it in `session_log.md` only. Do not pollute the avoid-list.
   - If the same call fails again in a future session and is still in the log but not promoted, surface it again: "This is the Nth failure of `<call>`. Promote now?"

### Step 4 — Success capture

When a non-trivial call **succeeds after prior struggle** (i.e. it's listed in the avoid-list, or you tried 2+ variations before this one worked), write the working signature to the relevant `tools/*.md` under `✅ Verified Working`. Be specific — exact argument types, version constraints if known.

Don't log every successful call. Only ones where future-Genie would benefit from knowing "this exact pattern works."

### Step 5 — Periodic consolidation

When a `tools/*.md` file exceeds ~200 lines, or after every ~20 sessions (whichever comes first), consolidate:

- Merge duplicate entries.
- Promote stable patterns from individual tool files to `patterns/working_recipes.md`.
- Move anti-patterns that span multiple tools to `patterns/anti_patterns.md`.
- Update `INDEX.md` so it stays a single screen.

Tell the user before consolidating: "I'm going to compress the memory files — N entries → M. Diff at the end if you want to review."

## File schemas

All schemas live in `references/schemas.md`. Genie should read that file when:

- Creating a new entry in any memory file
- Bootstrapping `.genie_memory/` for the first time
- Consolidating files

The schemas are strict — Genie parses these files in future sessions, so format drift breaks the skill.

## Bootstrap

If `.genie_memory/` doesn't exist, create the structure by copying `references/templates/` into the DBFS location. The templates are intentionally minimal — empty sections with the right headers. Do not pre-populate with guesses about what fails; the memory must be earned, not assumed.

```python
import os, shutil
from pathlib import Path

memory_root = Path(f"/dbfs/Users/{user}/.genie_memory")
if not memory_root.exists():
    template_root = Path("<path-to-skill>/references/templates")
    shutil.copytree(template_root, memory_root)
```

## What this skill does NOT do

- It does not auto-fix code. It records and warns.
- It does not promote failures to the avoid-list without user confirmation.
- It does not store secrets, tokens, full stack traces with PII, or query results. Only call signatures, error types, and short reasons.
- It does not replace good error handling in the actual code Genie writes — it's a layer above that.

## Reference files

- `references/schemas.md` — exact markdown schemas for every memory file. **Read this before creating or updating any memory file.**
- `references/templates/` — empty starter files copied into DBFS on first run.
- `references/common_failures.md` — a curated list of known Databricks SDK / API gotchas Genie can use as a starting reference (read-only, not part of memory).
