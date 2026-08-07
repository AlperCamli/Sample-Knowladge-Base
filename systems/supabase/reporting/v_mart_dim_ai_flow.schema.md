---
doc_class: machine-object
object: supabase.reporting.v_mart_dim_ai_flow
kind: view
schema_hash: "sha256:41453ba96d2c936e3834fa8bcb5483ba484aa2b7960589ed90a09780386b6d0b"
generated_at: 2026-08-04
source_mode: live
snapshot_version: "1"
status: machine
---

# `supabase.reporting.v_mart_dim_ai_flow`

## Identity

| Fact | Value |
|---|---|
| Object | `supabase.reporting.v_mart_dim_ai_flow` |
| Kind | view |
| Schema hash | `sha256:41453ba96d2c936e3834fa8bcb5483ba484aa2b7960589ed90a09780386b6d0b` |

## Columns

| # | Column | Type | Nullable | Default | Description | Purpose |
|---|---|---|---|---|---|---|
| 1 | `flow_type` | `text` | true | — | — | Logical AI flow executed; CHECK-constrained on `public.ai_runs` to 13 values. |
| 2 | `provider` | `text` | true | — | — | AI provider that served the runs in this group. |
| 3 | `model_name` | `text` | true | — | — | Specific model used. |
| 4 | `status` | `text` | true | — | — | Run status; CHECK-constrained on `public.ai_runs` to `pending`, `completed`, `failed`. |
| 5 | `run_count` | `bigint` | true | — | — | Runs matching this flow/provider/model/status combination. |
| 6 | `distinct_users` | `bigint` | true | — | — | Users with at least one run in this group; not additive across rows — see Warnings. |
| 7 | `input_tokens` | `bigint` | true | — | — | Sum of `ai_runs.input_tokens` across the group. |
| 8 | `output_tokens` | `bigint` | true | — | — | Sum of `ai_runs.output_tokens` across the group. |
| 9 | `total_tokens` | `bigint` | true | — | — | Sum of `ai_runs.total_tokens` across the group — the figure to use for cost. |
| 10 | `avg_seconds` | `numeric` | true | — | — | Mean `completed_at - started_at` in seconds to 2dp; always null for `pending` — see Warnings. |

## Keys & indexes

Primary key: —

Foreign keys: —

Unique constraints: —

Indexes: —

## Row estimate & freshness

Row estimate: —

Freshness: facts reflect the snapshot recorded in `generated_at` (front-matter).

## Referenced-by

—

## View definition

```sql
 SELECT flow_type,
    provider,
    model_name,
    status,
    run_count,
    distinct_users,
    input_tokens,
    output_tokens,
    total_tokens,
    avg_seconds
   FROM reporting.v_ai_runs_by_flow;
```
