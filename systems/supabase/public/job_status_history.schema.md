---
doc_class: machine-object
object: supabase.public.job_status_history
kind: table
schema_hash: "sha256:29df711db43f899a6d767c1a68ef6f404ed78e2667c0d0551adc1d3f61e06e5c"
generated_at: 2026-08-04
source_mode: live
snapshot_version: "1"
status: machine
---

# `supabase.public.job_status_history`

## Identity

| Fact | Value |
|---|---|
| Object | `supabase.public.job_status_history` |
| Kind | table |
| Schema hash | `sha256:29df711db43f899a6d767c1a68ef6f404ed78e2667c0d0551adc1d3f61e06e5c` |

## Columns

| # | Column | Type | Nullable | Default | Description | Purpose |
|---|---|---|---|---|---|---|
| 1 | `id` | `uuid` | false | `gen_random_uuid()` | — | Internal status-history entry id. |
| 2 | `job_id` | `uuid` | false | — | — | Job whose status transition is recorded (FK to jobs.id). |
| 3 | `from_status` | `text` | true | — | — | Previous status before the transition, or null for the first entry; enum in body. |
| 4 | `to_status` | `text` | false | — | — | New status after the transition; enum in body. |
| 5 | `changed_at` | `timestamp with time zone` | false | `now()` | — | When the transition was recorded. |
| 6 | `changed_by_user_id` | `uuid` | false | — | — | User who performed the transition (FK to users.id). |

## Keys & indexes

Primary key: `id`

Foreign keys:

| Columns | References | Referenced columns |
|---|---|---|
| `changed_by_user_id` | [`public.users`](users.schema.md) | `id` |
| `job_id` | [`public.jobs`](jobs.schema.md) | `id` |

Unique constraints: —

Indexes:

- `CREATE INDEX job_status_history_changed_by_user_id_idx ON public.job_status_history USING btree (changed_by_user_id, changed_at DESC)`
- `CREATE INDEX job_status_history_job_id_changed_at_idx ON public.job_status_history USING btree (job_id, changed_at DESC)`

Check constraints:

- `CHECK ((from_status = ANY (ARRAY['saved'::text, 'applied'::text, 'interview'::text, 'offer'::text, 'rejected'::text, 'archived'::text])) OR from_status IS NULL)`
- `CHECK (to_status = ANY (ARRAY['saved'::text, 'applied'::text, 'interview'::text, 'offer'::text, 'rejected'::text, 'archived'::text]))`

## Row estimate & freshness

Row estimate: —

Freshness: facts reflect the snapshot recorded in `generated_at` (front-matter).

## Referenced-by

—
