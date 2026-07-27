---
doc_class: human-object
object: supabase.reporting.v_ai_runs_by_day
written_against_schema_hash: "sha256:befbb2a86c14fc5684b9145b937e7664678abcaae081fce34aa8506d08cef9fd"
status: draft
last_verified: null
purpose: "AI runs started per UTC day, split by terminal status — the reliability trend series."
column_purposes:
  run_day: "UTC calendar date the run *started* (`ai_runs.started_at`), not the date it finished."
  status: "Run status, DB-constrained to `pending` | `completed` | `failed`. Enum in body."
  run_count: "Runs started that day with that status."
sources:
  - "platform: deploy/reporting-views.sql (view definition, CP-7 task 7.0 / D-81)"
  - "app DDL: public.ai_runs CHECK ai_runs_status_check + ai_runs_completion_consistency_check (read from the estate catalog, 2026-07-27)"
  - "machine doc: supabase.reporting.v_ai_runs_by_day"
  - "human doc: supabase.public.ai_runs"
depends_on:
  - supabase.public.ai_runs
contamination: null
---

# `supabase.reporting.v_ai_runs_by_day`

## Grain

One row per (UTC start date, status). A run appears exactly once, under the
day it started and its status at snapshot time.

## Column meanings & enum decodings

- `status` — **DB-constrained to `pending` | `completed` | `failed`**
  (`ai_runs_status_check`). A companion CHECK
  (`ai_runs_completion_consistency_check`) ties the vocabulary to the clock:
  `pending` rows have a null `completed_at`, and `completed`/`failed` rows
  must have one. So `pending` here means genuinely in flight (or abandoned
  without ever terminating), never "finished, outcome unknown".

## Reporting notes

- Daily failure rate is derived by the consumer, not baked into the view:
  `failed / (completed + failed)` per day, deciding explicitly whether
  `pending` belongs in the denominator. It usually does not — an in-flight
  run has not had the chance to fail yet.
- Splitting by status rather than filtering to failures keeps the total
  volume in the same result, so a rate and its base are read together. A
  failure percentage on two runs is not the same claim as one on two hundred.
- Per-flow and per-model breakdowns are a different view
  (`reporting.v_ai_runs_by_flow`); this one is deliberately time-first.

## Warnings

- **The bucket is the start date.** A run beginning at 23:58 and failing
  after midnight is counted on the day it started. Daily failure counts are
  therefore attributed to when work was requested, not when it went wrong.
- **`pending` is not a terminal state**, so the same day re-queried later can
  show a different split as runs finish. Only closed days are stable, and
  even they can move if a run hangs and is later resolved.
- Aggregate view over an RLS-protected table, evaluated as its owner; only a
  date, a status and a count are exposed — no user, no payload, no prompt.
  `ai_runs` holds free-text prompts and JSONB payloads, and none of it is
  reachable from here.
- Counts are small on this estate; a single failed run can look like a large
  percentage swing. Quote the absolute count alongside any rate.
