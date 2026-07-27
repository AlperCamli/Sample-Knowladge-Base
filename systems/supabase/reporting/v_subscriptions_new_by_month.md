---
doc_class: human-object
object: supabase.reporting.v_subscriptions_new_by_month
written_against_schema_hash: "sha256:a3fb70e6c96b954a9ea11e1935eb412a3807b9778d8b4d68e6dc6866f6bdb272"
status: draft
last_verified: null
purpose: "Subscription records created per UTC month and status — counts of records, never revenue."
column_purposes:
  month: "First day of the UTC month the subscription record was created in."
  status: "Lifecycle status of the record as it stands now, not as it was at creation. Open vocabulary — see Warnings."
  new_subscriptions: "Subscription records created that month with that current status."
sources:
  - "platform: deploy/reporting-views.sql (view definition, CP-7 task 7.0 / D-81)"
  - "machine doc: supabase.reporting.v_subscriptions_new_by_month"
  - "human doc: supabase.public.subscriptions"
  - "entity doc: conversion (subscriptions carry no amount column)"
depends_on:
  - supabase.public.subscriptions
contamination: null
---

# `supabase.reporting.v_subscriptions_new_by_month`

## Grain

One row per (UTC month of `created_at`, current `status`). The unit is the
**subscription record**, not the subscriber: a user accumulates a new row on
renewal, plan change or re-subscribe, so a month's total counts records
created, and the same person can appear in several months.

## Reporting notes

- Built to sit beside GA4's purchase count in a source-reconciliation
  report. The two will not agree exactly and are not meant to: this counts
  rows written to the billing table, GA4 counts events fired in the browser.
- Subscriber counts by plan are a different question and belong to
  `reporting.v_subscriptions_by_plan`, which is grouped on `plan_code`.

## Warnings

- **`status` is not a closed enum.** The database CHECK-constrains status on
  several tables, but **not this one** — verified against the estate catalog
  on 2026-07-27, no `subscriptions_status_check` exists. Grounded values are
  `active` and `trialing` (both named by the partial unique index that
  enforces one active subscription per user) and `canceled` (customer model,
  as an example rather than an enumeration). Provider-driven values such as
  `past_due` or `incomplete` may appear and are **not grounded here**. Any
  report that counts "active subscriptions" must pin the exact status values
  it means and say so.
- **`status` is current, `month` is historical.** A row created in January
  and cancelled in June appears under January with status `canceled`. The
  series is therefore not a stable record of what was true at the time, and
  re-running the same report later can move counts between status buckets
  within the same month. Do not treat it as a frozen cohort.
- **No revenue is available.** `public.subscriptions` has no amount, currency
  or price column, so nothing here can be turned into money without an
  external source. A "revenue by month" question is a gap, not a calculation.
- Aggregate view over an RLS-protected table (owner-evaluated, as with the
  rest of `reporting`); counts on an estate of roughly two dozen users are
  small enough that a single cell can be identifying.
