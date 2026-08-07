---
doc_class: metric
status: verified
last_verified: "2026-08-08 (Alper Camli)"
owner: "alper (operator)"
aliases: [AI error rate, run failure rate, AI reliability]
sources:
  - "customer-verified SQL: benchmark suite case RB-09 (.contextlayer/benchmark/suite.yaml)"
  - "machine doc: supabase.reporting.v_ai_runs_by_day (view definition)"
  - "app DDL: ai_runs_status_check — CHECK (status = ANY (ARRAY['pending','completed','failed'])), captured in snapshot supabase.json 2026-08-04"
  - "human doc: supabase.public.ai_runs"
depends_on:
  - supabase.public.ai_runs
  - supabase.reporting.v_ai_runs_by_day
contamination: null
---

# AI run failure rate

## Definition

**The share of AI runs started on a day that ended in `failed`.** Runs are
bucketed by `started_at`, not by when they finished, so a day's denominator is
stable the moment the day closes while its numerator can still move as
in-flight runs terminate.

## Formula

`failed_runs / total_runs` per UTC day, where a failure is `status = 'failed'`.

## Implementations

**`supabase` — base table.** Verbatim from RB-09:

```sql
SELECT date_trunc('day', started_at AT TIME ZONE 'UTC')::date AS run_day,
       count(*)                                         AS total_runs,
       count(*) FILTER (WHERE status = 'failed')        AS failed_runs,
       round(100.0 * count(*) FILTER (WHERE status = 'failed')
             / nullif(count(*), 0), 1)                  AS failure_pct
FROM public.ai_runs
WHERE started_at >= '<window_start>'
  AND started_at <  '<window_end>'
GROUP BY 1
ORDER BY 1;
```

**`supabase` — reporting view** (`reporting.v_ai_runs_by_day`, the RLS route),
which is split by status rather than pre-divided:

```sql
SELECT run_day,
       sum(run_count)                                        AS total_runs,
       sum(run_count) FILTER (WHERE status = 'failed')       AS failed_runs
FROM reporting.v_ai_runs_by_day
WHERE run_day >= '<window_start>' AND run_day < '<window_end>'
GROUP BY run_day ORDER BY run_day;
```

## Grain & dimensions

One row per UTC start day. Sliceable by flow through
`reporting.v_ai_runs_by_flow` (`flow_type`, `provider`, `model_name`), which is
the view to use when the question is *which flow* is failing.

## Known discrepancies

- **`pending` in the denominator.** The golden divides failures by *all* runs
  started, including ones still in flight, so a day queried early reads a lower
  failure rate than the same day queried later. Deciding whether `pending`
  belongs in the denominator is the report author's call and should be stated
  in the artifact.
- **A caveat the seed packet carried and the estate has since closed:** RB-09's
  note said `ai_runs.status` was text with no DB CHECK, so `'failed'` rested on
  an unconfirmed vocabulary. The 2026-08-04 snapshot captured
  `CHECK (status = ANY (ARRAY['pending','completed','failed']))` — the value is
  now DB-enforced and this metric's filter is grounded.
- Counts on this estate are small; one failed run can look like a large swing.
