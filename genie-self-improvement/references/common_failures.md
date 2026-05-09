# Common Databricks Failures (Reference)

_This is a curated starter list of well-known Databricks SDK / API gotchas. It's read-only and not part of the user's earned memory. Genie may consult it when stuck, but should still confirm with the user before adding any of these to the avoid-list._

## databricks-sdk-python

### `WorkspaceClient` auth in notebooks vs jobs
- In a notebook, `WorkspaceClient()` with no args picks up the runtime credentials.
- In a job, the same code may fail if the job runs as a different identity. Pass an explicit `Config` or use `notebook_context()`.

### `jobs.run_now(job_id, notebook_params=...)` with nested dicts
- `notebook_params` must be `dict[str, str]`. Nested values are silently rejected or truncated. Serialize to JSON string first.

### `clusters.list()` returns iterator, not list
- `len(w.clusters.list())` raises `TypeError`. Wrap in `list(...)` first.

### `statement_execution.execute_statement()` without `wait_timeout`
- Defaults to `10s`, then returns a pending statement. For sync behavior, set `wait_timeout="50s"` (max) and check `.status.state`.

### `files.upload()` path semantics
- For Unity Catalog volumes, path must start with `/Volumes/<catalog>/<schema>/<volume>/`.
- For DBFS, `/dbfs/...` (with leading slash) on the local filesystem, but `dbfs:/...` for SDK calls. Easy to mix up.

## REST API v2.0

### Token in header, not URL
- Always `Authorization: Bearer <token>`. Tokens in query strings get logged.

### `/api/2.0/jobs/run-now` payload is flat
- Same gotcha as SDK: `notebook_params` is flat `{str: str}`.

### Pagination uses `next_page_token`, not offset
- `/api/2.0/jobs/list` and similar return `has_more` + `next_page_token`. Don't try `offset=` — silently ignored.

### Endpoint deprecation
- `/api/2.0/dbfs/*` still works but is discouraged in UC-enabled workspaces. Prefer `/api/2.0/fs/files` (v2.0 unified files API) or Volumes.

## PySpark on Databricks

### `spark.sql(f"...{var}...")` SQL injection footgun
- Use `spark.sql("...", args={"var": value})` (DBR 14.1+) or parameterized queries via `pyspark.sql.functions.lit`.

### `display()` only works in notebooks
- Outside the notebook context (jobs, scripts), `display()` raises `NameError`. Use `df.show()` or write to table.

### `.toPandas()` on large DataFrames
- Drives OOM on driver. For >1M rows, use `.toPandas()` only after `.limit()`, or write to a Delta table and read with `pandas-on-spark`.

## Unity Catalog

### Three-part naming required
- `catalog.schema.table` everywhere. `schema.table` works only if a default catalog is set, and breaks reproducibility.

### Permissions vs ownership
- Granting `SELECT` on a table doesn't grant `USE SCHEMA` on the parent. UC requires the full chain: `USE CATALOG` → `USE SCHEMA` → table-level grants.

### Volumes vs DBFS
- New code: use Volumes. DBFS root is being phased out for UC workspaces. Mounts (`/mnt/...`) don't exist in UC-only mode.

## MLflow

### `mlflow.set_registry_uri("databricks-uc")` for UC model registry
- Without this, models register to the workspace registry, not Unity Catalog. Different APIs, different governance.

### Model name format in UC
- Must be `catalog.schema.model_name`. Workspace registry accepts a single string — easy to copy old code that breaks.

### `log_model` with custom code dependencies
- Use `code_paths=[...]` to bundle custom modules. Otherwise `load_model` in a different cluster fails with `ModuleNotFoundError`.

---

_When any of these match a real failure the user encounters, copy the relevant entry into the actual memory file (`tools/<tool>.md`) following the schema in `schemas.md`. Don't pre-populate memory from this list — only promote on real, observed failures._
