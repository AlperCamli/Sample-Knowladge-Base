---
doc_class: human-object
object: supabase.reporting.v_activation_funnel_monthly
written_against_schema_hash: "sha256:d1c38c2fe7c88702255c00612304dc05b7cc290661e4cf3a366e043a3f16bcf5"
status: draft
last_verified: null
purpose: "Monthly signup cohorts with same-month activation counts across four product milestones."
column_purposes:
  cohort_month: "First day of the UTC month the cohort's users signed up in."
  signed_up: "Users who signed up that month — the cohort size and the funnel's base."
  created_master_cv: "Cohort users with at least one non-deleted master CV created before the month ended."
  created_tailored_cv: "Cohort users with at least one non-deleted tailored CV created before the month ended."
  exported: "Cohort users with at least one completed export created before the month ended."
  subscribed: "Cohort users with at least one subscription record created before the month ended, at any status."
sources:
  - "platform: deploy/reporting-views.sql (view definition, CP-7 task 7.0 / D-81)"
  - "app DDL: public.exports CHECK exports_status_check (read from the estate catalog, 2026-07-27)"
  - "machine doc: supabase.reporting.v_activation_funnel_monthly"
  - "human doc: supabase.public.users"
  - "human doc: supabase.public.master_cvs"
  - "human doc: supabase.public.tailored_cvs"
  - "human doc: supabase.public.exports"
  - "human doc: supabase.public.subscriptions"
depends_on:
  - supabase.public.users
  - supabase.public.master_cvs
  - supabase.public.tailored_cvs
  - supabase.public.exports
  - supabase.public.subscriptions
contamination: null
---

# `supabase.reporting.v_activation_funnel_monthly`

## Grain

One row per UTC signup month. Every count is **users in that cohort**, never
rows of the underlying artefact: a user with nine tailored CVs adds one to
`created_tailored_cv`, not nine.

## What "same-month activation" means

A cohort user counts toward a stage if at least one qualifying row exists
**strictly before the end of their signup month**. The window is the cohort's
own month, not a rolling 30 days and not all time, which is what makes a
closed month's numbers final rather than drifting upward forever.

Stage predicates, exactly as the view applies them:

- `created_master_cv` / `created_tailored_cv` — a CV row owned by the user
  with `is_deleted = false`.
- `exported` — an export row owned by the user with `status = 'completed'`
  (DB-constrained to `processing` | `completed` | `failed`, so `completed`
  is a grounded value and not a convention).
- `subscribed` — **any** subscription row owned by the user, whatever its
  status. A trial that never converted, or one already cancelled, counts.

This view is the only one in `reporting` that joins rows internally. That is
the reason it exists: the execution role cannot perform this join itself,
because the base tables are RLS-protected and it reads nothing from them.

## Warnings

- **The stages are independent predicates, not a nested funnel.** Nothing
  enforces `exported ≤ created_tailored_cv`. A user who exported a CV and
  later soft-deleted it still counts in `exported` while dropping out of the
  CV stages, so counts can rise as you move rightward. Do not present it as a
  monotonic drop-off, and do not compute a "conversion rate" between two
  stages without saying that both are same-month, independent measures.
- **The current month is incomplete by construction.** Its activation window
  is still open, so the newest row always understates every stage except
  `signed_up`. Charts should mark it or exclude it.
- **`subscribed` is not "paying".** With any-status matching and a
  subscriptions vocabulary the database does not constrain, this column
  measures that a billing record appeared. Revenue is not available anywhere
  in this estate (no amount column).
- **Soft deletes rewrite history.** The CV stages filter on current
  `is_deleted`, not on whether the CV existed during the cohort month, so a
  deletion today lowers a closed month's number tomorrow. A stored report and
  a fresh run can legitimately disagree.
- Aggregate view over RLS-protected tables, evaluated as its owner. Cohorts
  on this estate run to single digits, and a stage count of 1 or 2 combined
  with a known signup month is potentially identifying — treat small cells as
  sensitive rather than reportable.
