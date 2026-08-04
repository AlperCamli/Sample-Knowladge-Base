---
doc_class: machine-object
object: supabase.public.exports
kind: table
schema_hash: "sha256:6c1cbfd40e67c8a748dd664e6087a626315612421b5660eda1f7d3d56c394316"
generated_at: 2026-08-04
source_mode: live
snapshot_version: "1"
status: machine
---

# `supabase.public.exports`

## Identity

| Fact | Value |
|---|---|
| Object | `supabase.public.exports` |
| Kind | table |
| Schema hash | `sha256:6c1cbfd40e67c8a748dd664e6087a626315612421b5660eda1f7d3d56c394316` |

## Columns

| # | Column | Type | Nullable | Default | Description | Purpose |
|---|---|---|---|---|---|---|
| 1 | `id` | `uuid` | false | `gen_random_uuid()` | — | Internal export id. |
| 2 | `user_id` | `uuid` | false | — | — | User that initiated the export (FK to users.id). |
| 3 | `tailored_cv_id` | `uuid` | true | — | — | Tailored CV being exported, when scope is tailored (FK). |
| 4 | `file_id` | `uuid` | true | — | — | Generated file artifact, populated when the export completes (FK to files.id). |
| 5 | `format` | `text` | false | — | — | Output format; enum in body. |
| 6 | `status` | `text` | false | — | — | Export lifecycle stage; enum in body. |
| 7 | `template_id` | `uuid` | true | — | — | Template used to render this export (FK to cv_templates.id). |
| 8 | `error_message` | `text` | true | — | — | Error description recorded when the export fails. |
| 9 | `created_at` | `timestamp with time zone` | false | `now()` | — | When the export was requested. |
| 10 | `completed_at` | `timestamp with time zone` | true | — | — | When the export reached a terminal state. |
| 11 | `master_cv_id` | `uuid` | true | — | — | Master CV being exported, when scope is master (FK). |

## Keys & indexes

Primary key: `id`

Foreign keys:

| Columns | References | Referenced columns |
|---|---|---|
| `file_id` | [`public.files`](files.schema.md) | `id` |
| `master_cv_id` | [`public.master_cvs`](master_cvs.schema.md) | `id` |
| `tailored_cv_id` | [`public.tailored_cvs`](tailored_cvs.schema.md) | `id` |
| `template_id` | [`public.cv_templates`](cv_templates.schema.md) | `id` |
| `user_id` | [`public.users`](users.schema.md) | `id` |

Unique constraints: —

Indexes:

- `CREATE INDEX exports_master_cv_id_created_at_idx ON public.exports USING btree (master_cv_id, created_at DESC) WHERE (master_cv_id IS NOT NULL)`
- `CREATE INDEX exports_status_created_at_idx ON public.exports USING btree (status, created_at DESC)`
- `CREATE INDEX exports_tailored_cv_id_created_at_idx ON public.exports USING btree (tailored_cv_id, created_at DESC)`
- `CREATE INDEX exports_user_id_created_at_idx ON public.exports USING btree (user_id, created_at DESC)`

Check constraints:

- `CHECK (format = ANY (ARRAY['pdf'::text, 'docx'::text]))`
- `CHECK (status = 'processing'::text AND file_id IS NULL AND completed_at IS NULL AND error_message IS NULL OR status = 'completed'::text AND file_id IS NOT NULL AND completed_at IS NOT NULL AND error_message IS NULL OR status = 'failed'::text AND file_id IS NULL AND completed_at IS NULL AND error_message IS NOT NULL)`
- `CHECK (status = ANY (ARRAY['processing'::text, 'completed'::text, 'failed'::text]))`
- `CHECK (tailored_cv_id IS NOT NULL AND master_cv_id IS NULL OR tailored_cv_id IS NULL AND master_cv_id IS NOT NULL)`

## Row estimate & freshness

Row estimate: 177

Freshness: facts reflect the snapshot recorded in `generated_at` (front-matter).

## Referenced-by

—
