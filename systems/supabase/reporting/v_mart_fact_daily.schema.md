---
doc_class: machine-object
object: supabase.reporting.v_mart_fact_daily
kind: view
schema_hash: "sha256:a8696c23ef09704d42f087654ae2e293b41cdbb242782f287d087bda409bb6fe"
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
| Schema hash | `sha256:a8696c23ef09704d42f087654ae2e293b41cdbb242782f287d087bda409bb6fe` |

## Columns

| # | Column | Type | Nullable | Default | Description | Purpose |
|---|---|---|---|---|---|---|
| 1 | `day` | `date` | true | — | — | — |
| 2 | `new_users` | `bigint` | true | — | — | — |
| 3 | `jobs_created` | `bigint` | true | — | — | — |
| 4 | `cvs_tailored` | `bigint` | true | — | — | — |
| 5 | `exports_created` | `bigint` | true | — | — | — |
| 6 | `ai_runs_started` | `bigint` | true | — | — | — |
| 7 | `ai_completed` | `numeric` | true | — | — | — |
| 8 | `ai_failed` | `numeric` | true | — | — | — |
| 9 | `ai_pending` | `numeric` | true | — | — | — |
| 10 | `job_transitions` | `numeric` | true | — | — | — |

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
     LEFT JOIN reporting.v_user_signups_by_day s ON s.signup_date = d.day
     LEFT JOIN ai ON ai.run_day = d.day
     LEFT JOIN tr ON tr.changed_day = d.day;
```
