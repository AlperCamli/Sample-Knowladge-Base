---
doc_class: machine-object
object: supabase.reporting.v_mart_fact_monthly
kind: view
schema_hash: "sha256:f4cd71e8f2e30e95eb84b9b1a8495e27981c23c99f8f9c893eccd56c42c66356"
generated_at: 2026-08-04
source_mode: live
snapshot_version: "1"
status: machine
---

# `supabase.reporting.v_mart_fact_monthly`

## Identity

| Fact | Value |
|---|---|
| Object | `supabase.reporting.v_mart_fact_monthly` |
| Kind | view |
| Schema hash | `sha256:f4cd71e8f2e30e95eb84b9b1a8495e27981c23c99f8f9c893eccd56c42c66356` |

## Columns

| # | Column | Type | Nullable | Default | Description | Purpose |
|---|---|---|---|---|---|---|
| 1 | `month` | `date` | true | — | — | — |
| 2 | `days_in_month` | `bigint` | true | — | — | — |
| 3 | `active_days` | `bigint` | true | — | — | — |
| 4 | `jobs_created` | `numeric` | true | — | — | — |
| 5 | `cvs_tailored` | `numeric` | true | — | — | — |
| 6 | `exports_created` | `numeric` | true | — | — | — |
| 7 | `ai_runs_started` | `numeric` | true | — | — | — |
| 8 | `total_tokens` | `numeric` | true | — | — | — |
| 9 | `ai_non_completed` | `numeric` | true | — | — | — |
| 10 | `tailored_cvs` | `numeric` | true | — | — | — |
| 11 | `tailored_deleted` | `numeric` | true | — | — | — |
| 12 | `job_count` | `bigint` | true | — | — | — |
| 13 | `job_users` | `bigint` | true | — | — | — |
| 14 | `applied_count` | `bigint` | true | — | — | — |
| 15 | `new_subscriptions` | `numeric` | true | — | — | — |
| 16 | `cohort_users` | `numeric` | true | — | — | — |
| 17 | `onboarded` | `numeric` | true | — | — | — |
| 18 | `signed_up` | `bigint` | true | — | — | — |
| 19 | `created_master_cv` | `bigint` | true | — | — | — |
| 20 | `created_tailored_cv` | `bigint` | true | — | — | — |
| 21 | `exported` | `bigint` | true | — | — | — |
| 22 | `subscribed` | `bigint` | true | — | — | — |

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
 WITH spine AS (
         SELECT date_trunc('month'::text, v_daily_activity.day::timestamp with time zone)::date AS month,
            count(*) AS days_in_month,
            count(*) FILTER (WHERE (v_daily_activity.jobs_created + v_daily_activity.cvs_tailored + v_daily_activity.exports_created + v_daily_activity.ai_runs_started) > 0) AS active_days,
            sum(v_daily_activity.jobs_created) AS jobs_created,
            sum(v_daily_activity.cvs_tailored) AS cvs_tailored,
            sum(v_daily_activity.exports_created) AS exports_created,
            sum(v_daily_activity.ai_runs_started) AS ai_runs_started
           FROM reporting.v_daily_activity
          GROUP BY (date_trunc('month'::text, v_daily_activity.day::timestamp with time zone)::date)
        ), tok AS (
         SELECT v_ai_tokens_by_month.month,
            sum(v_ai_tokens_by_month.total_tokens) AS total_tokens,
            sum(v_ai_tokens_by_month.run_count) AS token_runs,
            sum(v_ai_tokens_by_month.non_completed_count) AS ai_non_completed
           FROM reporting.v_ai_tokens_by_month
          GROUP BY v_ai_tokens_by_month.month
        ), cv AS (
         SELECT v_cv_production.month,
            sum(v_cv_production.tailored_cv_count) AS tailored_cvs,
            sum(v_cv_production.deleted_count) AS tailored_deleted
           FROM reporting.v_cv_production
          GROUP BY v_cv_production.month
        ), sub AS (
         SELECT v_subscriptions_new_by_month.month,
            sum(v_subscriptions_new_by_month.new_subscriptions) AS new_subscriptions
           FROM reporting.v_subscriptions_new_by_month
          GROUP BY v_subscriptions_new_by_month.month
        ), coh AS (
         SELECT v_user_cohorts.signup_month,
            sum(v_user_cohorts.user_count) AS cohort_users,
            sum(v_user_cohorts.onboarded_count) AS onboarded
           FROM reporting.v_user_cohorts
          GROUP BY v_user_cohorts.signup_month
        )
 SELECT s.month,
    s.days_in_month,
    s.active_days,
    s.jobs_created,
    s.cvs_tailored,
    s.exports_created,
    s.ai_runs_started,
    t.total_tokens,
    t.ai_non_completed,
    cv.tailored_cvs,
    cv.tailored_deleted,
    j.job_count,
    j.distinct_users AS job_users,
    j.applied_count,
    sub.new_subscriptions,
    coh.cohort_users,
    coh.onboarded,
    f.signed_up,
    f.created_master_cv,
    f.created_tailored_cv,
    f.exported,
    f.subscribed
   FROM spine s
     LEFT JOIN tok t ON t.month = s.month
     LEFT JOIN cv ON cv.month = s.month
     LEFT JOIN reporting.v_jobs_by_month j ON j.month = s.month
     LEFT JOIN sub ON sub.month = s.month
     LEFT JOIN coh ON coh.signup_month = s.month
     LEFT JOIN reporting.v_activation_funnel_monthly f ON f.cohort_month = s.month;
```
