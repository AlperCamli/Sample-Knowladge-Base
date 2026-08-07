---
doc_class: metric
status: verified
last_verified: "2026-08-08 (Alper Camli)"
owner: "alper (operator)"
aliases: [exports, successful exports, documents downloaded]
sources:
  - "customer-verified SQL: benchmark suite case RB-07 (activation step 4) (.contextlayer/benchmark/suite.yaml)"
  - "machine doc: supabase.reporting.v_exports_by_format (view definition)"
  - "app DDL: exports_status_check — CHECK (status = ANY (ARRAY['processing','completed','failed']))"
  - "human doc: supabase.public.exports"
depends_on:
  - supabase.public.exports
  - supabase.reporting.v_exports_by_format
contamination: null
---

# completed exports

## Definition

**Export jobs that produced a file**, i.e. rows with `status = 'completed'`.
The estate's closest thing to "a user got their CV out", and the fourth step of
the activation funnel. `processing` and `failed` rows are attempts, not
outcomes, and are excluded.

## Formula

`count(*)` over `public.exports` where `status = 'completed'` in the window;
`count(DISTINCT user_id)` where the question is people rather than files.

## Implementations

**`supabase` — base table**, as used verbatim inside RB-07's step 4:

```sql
SELECT count(DISTINCT e.user_id)
FROM public.exports e
WHERE e.status = 'completed'
  AND e.created_at < '<window_end>';
```

**`supabase` — reporting view** (`reporting.v_exports_by_format`, the RLS
route), which carries both counts by format:

```sql
SELECT format, export_count, distinct_users
FROM reporting.v_exports_by_format
WHERE status = 'completed';
```

## Grain & dimensions

Base table: one row per export, so any window and any grain. View: one row per
`(format, status)` with **no time dimension at all** — it is a lifetime
breakdown and cannot answer "last month".

## Known discrepancies

- **Files vs people.** `export_count` counts jobs; `distinct_users` counts
  people. A user exporting the same CV three times moves one and not the other.
  Say which one a report means.
- `file_id` is null for non-completed rows, so an inner join to `files` drops
  in-flight and failed exports silently — a filter this metric already applies,
  but worth knowing when composing.
- **Cover-letter exports live in a different table**
  (`public.cover_letter_exports`) and are *not* counted here. A question about
  "exports" that means both needs the two summed, deliberately.
