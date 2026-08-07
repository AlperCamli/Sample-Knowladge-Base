---
doc_class: human-object
object: supabase.reporting.v_mart_fact_monthly
written_against_schema_hash: "sha256:f4cd71e8f2e30e95eb84b9b1a8495e27981c23c99f8f9c893eccd56c42c66356"
status: draft
last_verified: null
purpose: "One row per month combining in-month activity totals with signup-cohort outcomes — two different grains side by side, see Warnings before comparing columns."
column_purposes:
  month: "First day of the month, from the activity spine."
  days_in_month: "Spine days present in this month — not the calendar length; partial in the first and current months."
  active_days: "Spine days in the month with at least one job, tailored CV, export or AI run."
  jobs_created: "Job postings created during the month."
  cvs_tailored: "Tailored CVs created during the month."
  exports_created: "Export jobs created during the month."
  ai_runs_started: "AI runs started during the month."
  total_tokens: "Billed AI tokens across all providers for runs started in the month."
  ai_non_completed: "Runs started in the month whose status is anything other than `completed`."
  tailored_cvs: "Tailored CVs created in the month, counted from the CV-production view including soft-deleted ones."
  tailored_deleted: "How many of `tailored_cvs` are now flagged `is_deleted` — a current flag, not a deletion in that month."
  job_count: "Job postings created in the month, counted independently of `jobs_created` — see Warnings."
  job_users: "Distinct users who created a job that month; not additive across months."
  applied_count: "Jobs created that month that now have a non-null `applied_at` — current state, not an in-month event."
  new_subscriptions: "Subscription records created in the month, UTC-bucketed — see Warnings on the timezone split."
  cohort_users: "Users who signed up in the month, from the cohorts view — contradicts `signed_up`, see Warnings."
  onboarded: "Users in that signup cohort who have completed onboarding, as of now."
  signed_up: "Cohort size: users who signed up in the month, UTC-bucketed — the funnel's base."
  created_master_cv: "Cohort users with a non-deleted master CV created before the month ended."
  created_tailored_cv: "Cohort users with a non-deleted tailored CV created before the month ended."
  exported: "Cohort users with a completed export created before the month ended."
  subscribed: "Cohort users with any subscription record created before the month ended."
sources:
  - "machine doc: supabase.reporting.v_mart_fact_monthly (view definition, pg_get_viewdef canonical)"
  - "machine doc: source view definitions — v_daily_activity, v_ai_tokens_by_month, v_cv_production, v_jobs_by_month, v_subscriptions_new_by_month, v_user_cohorts, v_activation_funnel_monthly"
  - "human doc: supabase.public.subscriptions (0 rows observed 2026-08-07)"
  - "observed 2026-08-07: session `TimeZone` is `UTC` (`current_setting('TimeZone')`), so the local/UTC split below is currently latent"
  - "observed 2026-08-07: `count(*)` as `contextlayer_exec` returns `permission denied` on this view, while the same role reads its source views normally"
depends_on:
  - supabase.reporting.v_daily_activity
  - supabase.reporting.v_ai_tokens_by_month
  - supabase.reporting.v_cv_production
  - supabase.reporting.v_jobs_by_month
  - supabase.reporting.v_subscriptions_new_by_month
  - supabase.reporting.v_user_cohorts
  - supabase.reporting.v_activation_funnel_monthly
  - supabase.public.jobs
  - supabase.public.subscriptions
---

# `supabase.reporting.v_mart_fact_monthly`

## Grain

One row per month on the activity spine — `v_daily_activity` aggregated to
month — with six sources `LEFT JOIN`ed on. The spine inherits
`generate_series(min(jobs.created_at)::date, CURRENT_DATE, '1 day')`, so
months exist here only from the first job posting onward.

**The row carries two grains.** Columns 2–15 measure *what happened during
the month*, over all users. Columns 16–22 measure *the cohort of users who
signed up that month* and what that cohort went on to do. These describe
different populations and are only on the same row because both key on a
month.

## Column meanings & enum decodings

The cohort block splits by source:

- `cohort_users` / `onboarded` come from `v_user_cohorts`, keyed on the
  signup month.
- `signed_up` through `subscribed` come from `v_activation_funnel_monthly`,
  keyed on `cohort_month`. Each funnel step counts cohort members who
  reached that milestone **before the signup month ended** — a same-month
  activation measure, not a lifetime one. A user who exported in their
  third month is not counted in `exported`.

`applied_count`, `tailored_deleted` and `onboarded` all read a *current*
flag against a *historical* cohort. They change retroactively as rows are
updated, so a month's figure is not stable across reruns.

## Reporting notes

- `active_days` over `days_in_month` gives an activity-density ratio, but
  only for whole months — see the partial-month warning.
- For AI cost by month use `total_tokens`; it already sums across providers.
  Split by provider requires `v_ai_tokens_by_month` directly.

## Warnings

- **Do not compute rates across the two grains.** `created_tailored_cv /
  cvs_tailored` looks like an activation rate and is not: the numerator
  counts users from one signup cohort, the denominator counts CVs made by
  everybody. Any ratio mixing columns 2–15 with 16–22 is meaningless.
- **The reporting role cannot read this view.** A `count(*)` as
  `contextlayer_exec` fails with `permission denied` (observed 2026-08-07),
  while the same role reads the source views normally. No `GRANT SELECT`
  reaches the execution role, so every claim here is derived from the view
  text rather than confirmed against rows.
- **`cohort_users` and `signed_up` measure the same thing on different
  calendars.** Both count users who signed up in the month, but
  `v_user_cohorts` buckets with a bare `date_trunc('month', created_at)` —
  resolved in the session `TimeZone` — while `v_activation_funnel_monthly`
  pins `AT TIME ZONE 'UTC'`. With the session at `UTC` (observed 2026-08-07)
  they agree; under any other setting a signup near a month boundary lands in
  a different month in each column. They are not interchangeable, and a
  mismatch is a timezone artefact rather than data loss. `signed_up` is the
  one that matches the rest of the funnel it heads.
- **The row mixes UTC and session-local months — a latent hazard.** UTC:
  `new_subscriptions`, `signed_up`, `created_master_cv`,
  `created_tailored_cv`, `exported`, `subscribed`. Session-local: everything
  else, including the `month` key itself. The session `TimeZone` was `UTC` on
  2026-08-07, so these coincide today. That is a configuration coincidence,
  not a property of the views — re-check `current_setting('TimeZone')` after
  any connection change before treating boundary months as comparable.
- **`days_in_month` is a spine count, not the calendar length.** The first
  month starts at the first job posting and the current month ends at
  `CURRENT_DATE`, so both are short. Do not use it to normalise a rate
  without excluding those two months, and do not assume 28–31.
- **The current month is always partial** across every column, and the
  cohort columns for it are additionally immature — a cohort's activation
  figures keep rising until its month closes.
- **`jobs_created` and `job_count` count the same objects by different
  paths** (daily spine sum vs. `v_jobs_by_month`). Both are session-local so
  they should agree; a divergence means one of the two source views has
  drifted and is worth investigating rather than averaging. The same applies
  to `cvs_tailored` against `tailored_cvs`.
- **`job_users` is not additive.** It is `count(DISTINCT user_id)` within a
  month; a user active in three months appears in all three. Summing months
  does not give distinct users for the period.
- **`new_subscriptions` is 0 or null throughout.** `public.subscriptions`
  held 0 rows on 2026-08-07, so this column and the `subscribed` funnel step
  report nothing today. A blank subscription trend is expected and is not a
  broken join.
- **An empty `public.jobs` empties the entire view**, spine and all — the
  same failure mode documented on `v_mart_fact_daily`.
