---
doc_class: human-object
object: supabase.reporting.v_job_status_transitions
written_against_schema_hash: "sha256:95c6baa1d34ce2f5709dff4f1feff12d5c5062114e6d1ad7cb45cf47a8855121"
status: contaminated
last_verified: null
purpose: "Daily counts of job-application status transitions — the source for a transition matrix."
column_purposes:
  changed_day: "UTC calendar date the transition was recorded (`job_status_history.changed_at`)."
  from_status: "Stage the job left; null marks a job's first status entry. Enum in body."
  to_status: "Stage the job moved to. Enum in body."
  transitions: "Transitions recorded that day for this (from, to) pair."
sources:
  - "platform: deploy/reporting-views.sql (view definition, CP-7 task 7.0 / D-81)"
  - "app DDL: public.job_status_history CHECK job_status_history_from_status_check / _to_status_check (read from the estate catalog, 2026-07-27)"
  - "machine doc: supabase.reporting.v_job_status_transitions"
  - "human doc: supabase.public.job_status_history"
depends_on:
  - supabase.public.job_status_history
contamination: {object: "supabase.public.job_status_history", change: "stat_changed", detail: "stat_changed: checks"}
---

# `supabase.reporting.v_job_status_transitions`

## Grain

One row per (UTC date, `from_status`, `to_status`) combination observed in
the append-only transition log. A single job contributes one row per
transition it makes, so jobs are counted repeatedly across the series by
design — this measures movement, not jobs.

## Column meanings & enum decodings

- `to_status` — DB-constrained to **`saved` | `applied` | `interview` |
  `offer` | `rejected` | `archived`** (`job_status_history_to_status_check`).
- `from_status` — the same set, **or `null`** for a job's first entry
  (`job_status_history_from_status_check` permits null explicitly). Null is
  kept raw here; render it as something like `(initial)` downstream rather
  than dropping the rows, which would silently discard every job's entry
  into the pipeline.

## Reporting notes

- A transition matrix is `from_status` × `to_status` summed over the date
  range. Rows where `from_status` is null form the intake column.
- Backwards moves are legitimate in this vocabulary (a job can return to
  `applied` from `interview`); do not assume the matrix is upper-triangular.

## Warnings

- **Aggregate view over an RLS-protected table.** `public.job_status_history`
  enforces per-user RLS; this view answers by evaluating RLS as its owner,
  which is why it exposes only a date, two stage names and a count.
- **The actor is deliberately not exposed.** `changed_by_user_id` exists on
  the base table and is not carried here: this view is the aggregate journey,
  not who moved what. A question about a specific user's jobs cannot be
  answered from it, and should not be approximated from it.
- **Small counts can still identify** on an estate of roughly two dozen
  users — a single transition on a given day is potentially personal.
- The date is when the transition was **recorded**, not when the underlying
  real-world event happened; a batch of catch-up edits lands on the edit day.
