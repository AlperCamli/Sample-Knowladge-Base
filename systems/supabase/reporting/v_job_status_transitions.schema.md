---
doc_class: machine-object
object: supabase.reporting.v_job_status_transitions
kind: view
schema_hash: "sha256:95c6baa1d34ce2f5709dff4f1feff12d5c5062114e6d1ad7cb45cf47a8855121"
generated_at: 2026-07-27
source_mode: live
snapshot_version: "1"
status: machine
---

# `supabase.reporting.v_job_status_transitions`

## Identity

| Fact | Value |
|---|---|
| Object | `supabase.reporting.v_job_status_transitions` |
| Kind | view |
| Schema hash | `sha256:95c6baa1d34ce2f5709dff4f1feff12d5c5062114e6d1ad7cb45cf47a8855121` |

## Columns

| # | Column | Type | Nullable | Default | Description | Purpose |
|---|---|---|---|---|---|---|
| 1 | `changed_day` | `date` | true | — | — | UTC calendar date the transition was recorded (`job_status_history.changed_at`). |
| 2 | `from_status` | `text` | true | — | — | Stage the job left; null marks a job's first status entry. Enum in body. |
| 3 | `to_status` | `text` | true | — | — | Stage the job moved to. Enum in body. |
| 4 | `transitions` | `bigint` | true | — | — | Transitions recorded that day for this (from, to) pair. |

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
 SELECT (changed_at AT TIME ZONE 'UTC'::text)::date AS changed_day,
    from_status,
    to_status,
    count(*) AS transitions
   FROM public.job_status_history
  GROUP BY ((changed_at AT TIME ZONE 'UTC'::text)::date), from_status, to_status;
```
