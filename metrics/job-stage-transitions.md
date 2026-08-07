---
doc_class: metric
status: verified
last_verified: "2026-08-08 (Alper Camli)"
owner: "alper (operator)"
aliases: [application pipeline movement, stage transitions, job funnel moves]
sources:
  - "customer-verified SQL: benchmark suite case RB-06 (.contextlayer/benchmark/suite.yaml)"
  - "machine doc: supabase.reporting.v_job_status_transitions (view definition)"
  - "app DDL: job_status_history_to_status_check / _from_status_check (six-value set)"
  - "human doc: supabase.public.job_status_history"
depends_on:
  - supabase.public.job_status_history
  - supabase.reporting.v_job_status_transitions
contamination: null
---

# job stage transitions

## Definition

**Moves of a tracked job from one application stage to another**, counted from
the append-only history table. One row of `job_status_history` is one move; the
first entry for a job has `from_status = null` and represents intake rather
than a move.

## Formula

`count(*)` over `public.job_status_history` in the window, grouped by
`(from_status, to_status)`. The customer's golden renders `null` as
`'(initial)'` so the intake column is visible rather than dropped.

## Implementations

**`supabase` — base table.** Verbatim from RB-06:

```sql
SELECT coalesce(from_status, '(initial)') AS from_status,
       to_status,
       count(*) AS transitions
FROM public.job_status_history
WHERE changed_at >= '<window_start>'
  AND changed_at <  '<window_end>'
GROUP BY 1, 2
ORDER BY 1, 2;
```

**`supabase` — reporting view** (`reporting.v_job_status_transitions`, the RLS
route), which keeps the day and leaves `from_status` null:

```sql
SELECT from_status, to_status, sum(transitions) AS transitions
FROM reporting.v_job_status_transitions
WHERE changed_day >= '<window_start>' AND changed_day < '<window_end>'
GROUP BY from_status, to_status ORDER BY 1, 2;
```

## Grain & dimensions

One row per `(from_status, to_status)` pair, optionally per UTC day through the
view. Both stage columns are DB-constrained to
`saved | applied | interview | offer | rejected | archived`; `from_status` is
additionally nullable for a job's first entry.

## Known discrepancies

- **The matrix is not upper-triangular.** Backwards moves happen (`interview` →
  `applied`), and rendering the matrix as a one-way funnel hides them.
- **Time in stage is not this metric.** It comes from the same table by
  differencing `changed_at` per job, and nothing here computes it.
- Legacy spellings `interviewing`/`offered` were migrated before this table
  existed, so they do not appear here — but they may appear in
  `public.jobs.status` history elsewhere. See
  [`jobs.md`](../systems/supabase/public/jobs.md).
