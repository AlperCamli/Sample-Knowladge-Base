---
doc_class: machine-object
object: supabase.reporting.v_ai_runs_by_day
kind: view
schema_hash: "sha256:befbb2a86c14fc5684b9145b937e7664678abcaae081fce34aa8506d08cef9fd"
generated_at: 2026-07-25
source_mode: live
snapshot_version: "1"
status: machine
---

# `supabase.reporting.v_ai_runs_by_day`

## Identity

| Fact | Value |
|---|---|
| Object | `supabase.reporting.v_ai_runs_by_day` |
| Kind | view |
| Schema hash | `sha256:befbb2a86c14fc5684b9145b937e7664678abcaae081fce34aa8506d08cef9fd` |

## Columns

| # | Column | Type | Nullable | Default | Description | Purpose |
|---|---|---|---|---|---|---|
| 1 | `run_day` | `date` | true | — | — | — |
| 2 | `status` | `text` | true | — | — | — |
| 3 | `run_count` | `bigint` | true | — | — | — |

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
 SELECT (started_at AT TIME ZONE 'UTC'::text)::date AS run_day,
    status,
    count(*) AS run_count
   FROM public.ai_runs
  GROUP BY ((started_at AT TIME ZONE 'UTC'::text)::date), status;
```
