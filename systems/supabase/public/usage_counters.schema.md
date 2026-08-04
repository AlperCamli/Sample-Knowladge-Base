---
doc_class: machine-object
object: supabase.public.usage_counters
kind: table
schema_hash: "sha256:d6d645c999291cbdac6590455a6beae64eaa7519a9692d293239519458e64393"
generated_at: 2026-08-04
source_mode: live
snapshot_version: "1"
status: machine
---

# `supabase.public.usage_counters`

## Identity

| Fact | Value |
|---|---|
| Object | `supabase.public.usage_counters` |
| Kind | table |
| Schema hash | `sha256:d6d645c999291cbdac6590455a6beae64eaa7519a9692d293239519458e64393` |

## Columns

| # | Column | Type | Nullable | Default | Description | Purpose |
|---|---|---|---|---|---|---|
| 1 | `id` | `uuid` | false | `gen_random_uuid()` | — | Internal counter-row id. |
| 2 | `user_id` | `uuid` | false | — | — | User whose usage is tracked (FK to users.id). |
| 3 | `period_month` | `date` | false | — | — | First day of the calendar month this row covers. |
| 4 | `tailored_cv_generations_count` | `integer` | false | `0` | — | Tailored CV generations consumed in the period; free-tier cap 3/month, API-enforced. |
| 5 | `exports_count` | `integer` | false | `0` | — | Export jobs completed in the period; free-tier cap 5/month, API-enforced. |
| 6 | `ai_actions_count` | `integer` | false | `0` | — | AI block actions invoked in the period; together with tailored generations, capped at 20 combined AI uses/month on the free tier (API-enforced). |
| 7 | `storage_bytes_used` | `bigint` | false | `0` | — | Total bytes of files stored as of this period. |
| 8 | `updated_at` | `timestamp with time zone` | false | `now()` | — | When this counter was last incremented. |

## Keys & indexes

Primary key: `id`

Foreign keys:

| Columns | References | Referenced columns |
|---|---|---|
| `user_id` | [`public.users`](users.schema.md) | `id` |

Unique constraints:

- `user_id`, `period_month`

Indexes:

- `CREATE INDEX usage_counters_period_month_idx ON public.usage_counters USING btree (period_month)`
- `CREATE INDEX usage_counters_user_id_idx ON public.usage_counters USING btree (user_id)`

Check constraints:

- `CHECK (ai_actions_count >= 0)`
- `CHECK (exports_count >= 0)`
- `CHECK (period_month = date_trunc('month'::text, period_month::timestamp with time zone)::date)`
- `CHECK (storage_bytes_used >= 0)`
- `CHECK (tailored_cv_generations_count >= 0)`

## Row estimate & freshness

Row estimate: 45

Freshness: facts reflect the snapshot recorded in `generated_at` (front-matter).

## Referenced-by

—
