---
doc_class: human-object
object: supabase.reporting.v_mart_dim_ai_flow
written_against_schema_hash: "sha256:41453ba96d2c936e3834fa8bcb5483ba484aa2b7960589ed90a09780386b6d0b"
status: draft
last_verified: null
purpose: "Unfiltered pass-through of `v_ai_runs_by_flow` — the AI-flow dimension of the mart layer."
column_purposes:
  flow_type: "Logical AI flow executed; CHECK-constrained on `public.ai_runs` to 13 values."
  provider: "AI provider that served the runs in this group."
  model_name: "Specific model used."
  status: "Run status; CHECK-constrained on `public.ai_runs` to `pending`, `completed`, `failed`."
  run_count: "Runs matching this flow/provider/model/status combination."
  distinct_users: "Users with at least one run in this group; not additive across rows — see Warnings."
  input_tokens: "Sum of `ai_runs.input_tokens` across the group."
  output_tokens: "Sum of `ai_runs.output_tokens` across the group."
  total_tokens: "Sum of `ai_runs.total_tokens` across the group — the figure to use for cost."
  avg_seconds: "Mean `completed_at - started_at` in seconds to 2dp; always null for `pending` — see Warnings."
sources:
  - "machine doc: supabase.reporting.v_mart_dim_ai_flow (view definition, pg_get_viewdef canonical)"
  - "machine doc: supabase.reporting.v_ai_runs_by_flow (view definition, pg_get_viewdef canonical)"
  - "app DDL: public.ai_runs CHECK constraints on `flow_type`, `status`, and the status/`completed_at` pairing"
  - "observed 2026-08-07: `count(*)` as `contextlayer_exec` returns `permission denied for view v_mart_dim_ai_flow`, while the same role reads the source view (48 rows) — the grant is missing on the mart views specifically"
depends_on:
  - supabase.reporting.v_ai_runs_by_flow
  - supabase.public.ai_runs
---

# `supabase.reporting.v_mart_dim_ai_flow`

## Grain

One row per `(flow_type, provider, model_name, status)` combination — the
grain of `reporting.v_ai_runs_by_flow`, inherited unchanged.

The view is a bare `SELECT` of all ten columns of that view, with no filter,
no re-aggregation and no renaming. It is an alias, not a transformation: for
any query, this view and its source return byte-identical results.

## Reporting notes

- **This view adds no information over its source.** Prefer it only where a
  consumer wants the `v_mart_*` naming for a fact/dim layer; for anything
  else, `v_ai_runs_by_flow` is the same data with a documented lineage one
  hop shorter.
- There is no date column here. This dimension is cumulative over all history
  in `public.ai_runs` — it cannot be filtered to a period. For time-sliced AI
  volume use `v_mart_fact_daily` (started-run counts by day) or
  `v_ai_tokens_by_month` (token spend by month).

## Warnings

- **The reporting role cannot read this view.** A `count(*)` as
  `contextlayer_exec` fails with `permission denied for view
  v_mart_dim_ai_flow` (observed 2026-08-07), while the same role reads
  `v_ai_runs_by_flow` normally. The view exists in the snapshot and is
  introspectable, but no `GRANT SELECT` reaches the execution role, so every
  column description below is derived from the view text rather than from
  data. Until a DBA applies the grant, route queries to the source view.
- **`distinct_users` is not additive.** It is `count(DISTINCT user_id)`
  evaluated *within* each group, so a user who used three flows is counted
  once in each of those three rows. Summing the column across flows,
  providers or models double-counts, and there is no combination of these
  rows that yields a correct total user count. Compute that from
  `public.ai_runs` directly.
- **`avg_seconds` is always null when `status = 'pending'`.** The
  `ai_runs` CHECK constraint requires `completed_at IS NULL` for pending
  runs, so `avg(completed_at - started_at)` has nothing to average. A null
  here means "not finished", never "instant" — do not coalesce it to zero.
- **`avg_seconds` is a mean over unbounded values** with no count exposed
  alongside it per row other than `run_count`; a single stalled run that
  later completed can move a small group's mean substantially. Weight by
  `run_count` when aggregating across groups, and never average the averages.
- **The token sums ignore nulls.** `sum()` skips null token columns rather
  than propagating, so a group mixing recorded and unrecorded token counts
  reports the sum of the recorded ones with no indication that it is partial.
