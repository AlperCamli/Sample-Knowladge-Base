---
doc_class: machine-object
object: supabase.reporting.v_mart_fact_daily
kind: view
schema_hash: "sha256:dbc344ec4b5b1e7e43ebcf97ca4261bce7ce9457990af05494c91c0a29ef9351"
generated_at: 2026-08-04
source_mode: live
snapshot_version: "1"
status: machine
---

# `supabase.reporting.v_mart_fact_daily`

## Identity

| Fact | Value |
|---|---|
| Object | `supabase.reporting.v_mart_fact_daily` |
| Kind | view |
| Schema hash | `sha256:dbc344ec4b5b1e7e43ebcf97ca4261bce7ce9457990af05494c91c0a29ef9351` |

## Columns

| # | Column | Type | Nullable | Default | Description | Purpose |
|---|---|---|---|---|---|---|
| 1 | `day` | `date` | true | — | — | Spine date. Runs from the first job ever created to today — see Warnings, the spine is not the calendar. |
| 2 | `new_users` | `bigint` | true | — | — | Accounts created that day, UTC-bucketed; 0 where no signups matched — see Warnings on the timezone split. |
| 3 | `jobs_created` | `bigint` | true | — | — | Job postings created that day. |
| 4 | `cvs_tailored` | `bigint` | true | — | — | Tailored CVs created that day. |
| 5 | `exports_created` | `bigint` | true | — | — | Export jobs created that day. |
| 6 | `ai_runs_started` | `bigint` | true | — | — | AI runs started that day. |
| 7 | `ai_completed` | `numeric` | true | — | — | Runs *started* that day whose status is now `completed` — not completions on that day. |
| 8 | `ai_failed` | `numeric` | true | — | — | Runs *started* that day whose status is now `failed`. |
| 9 | `ai_pending` | `numeric` | true | — | — | Runs *started* that day still `pending` — a running total that shrinks as they finish. |
| 10 | `job_transitions` | `numeric` | true | — | — | Job-application status changes recorded that day. |

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
 WITH ai AS (
         SELECT v_ai_runs_by_day.run_day,
            COALESCE(sum(v_ai_runs_by_day.run_count) FILTER (WHERE v_ai_runs_by_day.status = 'completed'::text), 0::numeric) AS ai_completed,
            COALESCE(sum(v_ai_runs_by_day.run_count) FILTER (WHERE v_ai_runs_by_day.status = 'failed'::text), 0::numeric) AS ai_failed,
            COALESCE(sum(v_ai_runs_by_day.run_count) FILTER (WHERE v_ai_runs_by_day.status = 'pending'::text), 0::numeric) AS ai_pending
           FROM reporting.v_ai_runs_by_day
          GROUP BY v_ai_runs_by_day.run_day
        ), tr AS (
         SELECT v_job_status_transitions.changed_day,
            sum(v_job_status_transitions.transitions) AS job_transitions
           FROM reporting.v_job_status_transitions
          GROUP BY v_job_status_transitions.changed_day
        )
 SELECT d.day,
    COALESCE(s.new_users, 0::bigint) AS new_users,
    d.jobs_created,
    d.cvs_tailored,
    d.exports_created,
    d.ai_runs_started,
    COALESCE(ai.ai_completed, 0::numeric) AS ai_completed,
    COALESCE(ai.ai_failed, 0::numeric) AS ai_failed,
    COALESCE(ai.ai_pending, 0::numeric) AS ai_pending,
    COALESCE(tr.job_transitions, 0::numeric) AS job_transitions
   FROM reporting.v_daily_activity d
     LEFT JOIN reporting.v_user_signups_by_day s ON s.signup_day = d.day
     LEFT JOIN ai ON ai.run_day = d.day
     LEFT JOIN tr ON tr.changed_day = d.day;
```
