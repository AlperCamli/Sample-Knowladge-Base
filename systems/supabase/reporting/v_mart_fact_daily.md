---
doc_class: human-object
object: supabase.reporting.v_mart_fact_daily
written_against_schema_hash: "sha256:dbc344ec4b5b1e7e43ebcf97ca4261bce7ce9457990af05494c91c0a29ef9351"
status: draft
last_verified: null
purpose: "One row per day on the activity spine, joining signups, the four core actions, AI run outcomes and job transitions into a single daily fact row."
column_purposes:
  day: "Spine date. Runs from the first job ever created to today — see Warnings, the spine is not the calendar."
  new_users: "Accounts created that day, UTC-bucketed; 0 where no signups matched — see Warnings on the timezone split."
  jobs_created: "Job postings created that day."
  cvs_tailored: "Tailored CVs created that day."
  exports_created: "Export jobs created that day."
  ai_runs_started: "AI runs started that day."
  ai_completed: "Runs *started* that day whose status is now `completed` — not completions on that day."
  ai_failed: "Runs *started* that day whose status is now `failed`."
  ai_pending: "Runs *started* that day still `pending` — a running total that shrinks as they finish."
  job_transitions: "Job-application status changes recorded that day."
sources:
  - "machine doc: supabase.reporting.v_mart_fact_daily (view definition, pg_get_viewdef canonical)"
  - "machine doc: source view definitions — v_daily_activity, v_user_signups_by_day, v_ai_runs_by_day, v_job_status_transitions"
  - "app DDL: public.ai_runs CHECK (status = ANY (ARRAY['pending','completed','failed'])) — settles that the three AI columns exhaust the status space"
  - "observed 2026-08-07: session `TimeZone` is `UTC` (`current_setting('TimeZone')`), so the local/UTC split below is currently latent"
  - "observed 2026-08-07: `count(*)` as `contextlayer_exec` returns `permission denied` on this view, while the same role reads `v_daily_activity` (112 rows) and `v_user_signups_by_day` (23 rows)"
depends_on:
  - supabase.reporting.v_daily_activity
  - supabase.reporting.v_user_signups_by_day
  - supabase.reporting.v_ai_runs_by_day
  - supabase.reporting.v_job_status_transitions
  - supabase.public.jobs
  - supabase.public.ai_runs
---

# `supabase.reporting.v_mart_fact_daily`

## Grain

One row per date on the `v_daily_activity` spine. Everything else is a
`LEFT JOIN` onto that spine and is `COALESCE`d to 0, so every spine day
appears exactly once whether or not anything happened on it.

The spine is not a calendar. It is
`generate_series(min(jobs.created_at)::date, CURRENT_DATE, '1 day')` — it
begins on **the day the first job posting was ever created** and ends today.

## Column meanings & enum decodings

The three AI outcome columns pivot `v_ai_runs_by_day` by status. The
`public.ai_runs` CHECK constrains `status` to exactly `pending`,
`completed` and `failed`, so those three columns partition the runs
started that day with no fourth bucket hiding rows.

All three are keyed on the day the run **started**, not the day it reached
its status. `ai_completed` for a given day is therefore a property of that
day's cohort of runs as of query time, not a count of completion events —
and `ai_pending` for an old day is a backlog figure that shrinks over time
as those runs finish.

## Reporting notes

- A zero is a real zero for spine days: gaps are filled, so a line chart can
  be plotted directly without a calendar join.
- `ai_completed + ai_failed + ai_pending` should equal that day's AI runs —
  but read the timezone warning before reconciling it against
  `ai_runs_started`.

## Warnings

- **The reporting role cannot read this view.** A `count(*)` as
  `contextlayer_exec` fails with `permission denied` (observed 2026-08-07),
  while the same role reads `v_daily_activity` and `v_user_signups_by_day`
  normally. No `GRANT SELECT` reaches the execution role, so every claim
  below is derived from the view text rather than confirmed against rows.
  Until a DBA applies the grant, assemble the equivalent from the source
  views.
- **The columns are bucketed on two different calendars — a latent hazard.**
  `v_daily_activity` casts timestamps with a bare `::date`, which resolves in
  the session `TimeZone`; `v_user_signups_by_day` and `v_ai_runs_by_day` both
  pin `AT TIME ZONE 'UTC'`. So `day`, `jobs_created`, `cvs_tailored`,
  `exports_created` and `ai_runs_started` are **session-local**, while
  `new_users`, `ai_completed`, `ai_failed` and `ai_pending` are **UTC**. The
  session `TimeZone` was observed as `UTC` on 2026-08-07, so the two agree
  today and the join is sound. **This is a configuration coincidence, not a
  property of the views.** If the connection's `TimeZone` is ever changed,
  the two halves of every row silently start bucketing on different
  calendars, and `ai_runs_started` stops reconciling against the three AI
  outcome columns even though the CHECK constraint guarantees they are
  exhaustive. Nothing in the schema would flag that; re-check
  `current_setting('TimeZone')` before trusting this view after any
  connection change.
- **Activity before the first job posting is invisible.** The spine starts at
  `min(jobs.created_at)`, so a signup, CV or export that predates the first
  job is not merely zeroed — its day is absent from the view entirely, and
  its counts appear nowhere. Totals from this view are therefore not
  guaranteed to match totals taken from the base tables.
- **A `new_users` of 0 is ambiguous.** It means "no signups matched this
  spine day", which covers both a genuinely quiet day and a day whose
  signups fell outside the spine's range or on the other side of the UTC
  boundary above.
- **An empty `public.jobs` empties the whole view.** `min()` over no rows is
  null, `generate_series` from null yields no rows, and every column here
  hangs off that spine. The view would return zero rows while the underlying
  activity tables were still populated — a total outage that looks exactly
  like "no activity".
- **The spine ends at `CURRENT_DATE`, so today's row is always partial** and
  will keep rising until the day closes. Exclude it from trend comparisons.
