---
doc_class: machine-object
object: supabase.public.jobs
kind: table
schema_hash: "sha256:bc07750f3d955f87a42ed05d09865f43c26328ba940a2c80c3b6b7cf6504a856"
generated_at: 2026-08-04
source_mode: live
snapshot_version: "1"
status: machine
---

# `supabase.public.jobs`

## Identity

| Fact | Value |
|---|---|
| Object | `supabase.public.jobs` |
| Kind | table |
| Schema hash | `sha256:bc07750f3d955f87a42ed05d09865f43c26328ba940a2c80c3b6b7cf6504a856` |

## Columns

| # | Column | Type | Nullable | Default | Description | Purpose |
|---|---|---|---|---|---|---|
| 1 | `id` | `uuid` | false | `gen_random_uuid()` | — | Internal job id. |
| 2 | `user_id` | `uuid` | false | — | — | Owning user who saved this job (FK to users.id). |
| 3 | `tailored_cv_id` | `uuid` | true | — | — | Tailored CV currently associated with this job (FK to tailored_cvs.id). |
| 4 | `company_name` | `text` | false | — | — | Company posting the job. |
| 5 | `job_title` | `text` | false | — | — | Title of the role being applied to. |
| 6 | `job_description` | `text` | false | — | — | Full job description text used for AI tailoring and analysis. |
| 7 | `job_posting_url` | `text` | true | — | — | URL of the original posting, if known. |
| 8 | `location_text` | `text` | true | — | — | Free-text location for the role. |
| 9 | `status` | `text` | false | `'saved'::text` | — | Application pipeline stage; enum decoded in body. |
| 10 | `notes` | `text` | true | — | — | User's free-form notes about this job. |
| 11 | `applied_at` | `timestamp with time zone` | true | — | — | When the user marked the job as applied. |
| 12 | `created_at` | `timestamp with time zone` | false | `now()` | — | When the job record was created. |
| 13 | `updated_at` | `timestamp with time zone` | false | `now()` | — | When the job record was last updated. |
| 14 | `cover_letter_id` | `uuid` | true | — | — | Cover letter currently associated with this job (FK to cover_letters.id). |

## Keys & indexes

Primary key: `id`

Foreign keys:

| Columns | References | Referenced columns |
|---|---|---|
| `cover_letter_id` | [`public.cover_letters`](cover_letters.schema.md) | `id` |
| `tailored_cv_id` | [`public.tailored_cvs`](tailored_cvs.schema.md) | `id` |
| `user_id` | [`public.users`](users.schema.md) | `id` |

Unique constraints: —

Indexes:

- `CREATE INDEX jobs_company_name_idx ON public.jobs USING btree (company_name)`
- `CREATE INDEX jobs_status_idx ON public.jobs USING btree (status)`
- `CREATE INDEX jobs_tailored_cv_id_idx ON public.jobs USING btree (tailored_cv_id)`
- `CREATE INDEX jobs_user_id_idx ON public.jobs USING btree (user_id)`
- `CREATE UNIQUE INDEX jobs_unique_cover_letter_idx ON public.jobs USING btree (cover_letter_id) WHERE (cover_letter_id IS NOT NULL)`
- `CREATE UNIQUE INDEX jobs_unique_tailored_cv_idx ON public.jobs USING btree (tailored_cv_id) WHERE (tailored_cv_id IS NOT NULL)`

Check constraints:

- `CHECK (status = ANY (ARRAY['saved'::text, 'applied'::text, 'interview'::text, 'offer'::text, 'rejected'::text, 'archived'::text]))`

## Row estimate & freshness

Row estimate: 102

Freshness: facts reflect the snapshot recorded in `generated_at` (front-matter).

## Referenced-by

| Object | Via |
|---|---|
| [`supabase.public.ai_runs`](ai_runs.schema.md) | `job_id` |
| [`supabase.public.cover_letters`](cover_letters.schema.md) | `job_id` |
| [`supabase.public.job_status_history`](job_status_history.schema.md) | `job_id` |
| [`supabase.public.tailored_cvs`](tailored_cvs.schema.md) | `job_id` |
