---
doc_class: machine-object
object: supabase.public.files
kind: table
schema_hash: "sha256:a3a1531959d76aab8a389c584aa353dc06718b96a8df3979de7e3b5ee36c4f7a"
generated_at: 2026-07-27
source_mode: live
snapshot_version: "1"
status: machine
---

# `supabase.public.files`

## Identity

| Fact | Value |
|---|---|
| Object | `supabase.public.files` |
| Kind | table |
| Schema hash | `sha256:a3a1531959d76aab8a389c584aa353dc06718b96a8df3979de7e3b5ee36c4f7a` |

## Columns

| # | Column | Type | Nullable | Default | Description | Purpose |
|---|---|---|---|---|---|---|
| 1 | `id` | `uuid` | false | `gen_random_uuid()` | — | Internal file id (referenced by imports and exports). |
| 2 | `user_id` | `uuid` | false | — | — | Owning user this file belongs to (FK to users.id). |
| 3 | `file_type` | `text` | false | — | — | Domain role of the file; enum in body. |
| 4 | `storage_bucket` | `text` | false | — | — | Name of the storage bucket that holds the object. |
| 5 | `storage_path` | `text` | false | — | — | Path of the object inside the storage bucket. |
| 6 | `original_filename` | `text` | false | — | — | Original filename supplied at upload time. |
| 7 | `mime_type` | `text` | false | — | — | MIME type of the stored object. |
| 8 | `size_bytes` | `bigint` | false | — | — | Size of the stored object in bytes. |
| 9 | `checksum` | `text` | true | — | — | Optional content checksum for duplicate/corruption detection. |
| 10 | `is_deleted` | `boolean` | false | `false` | — | Soft-delete flag hiding the file from active listings. |
| 11 | `created_at` | `timestamp with time zone` | false | `now()` | — | When this file record was created. |

## Keys & indexes

Primary key: `id`

Foreign keys:

| Columns | References | Referenced columns |
|---|---|---|
| `user_id` | [`public.users`](users.schema.md) | `id` |

Unique constraints: —

Indexes:

- `CREATE INDEX files_file_type_idx ON public.files USING btree (file_type)`
- `CREATE INDEX files_user_id_is_deleted_idx ON public.files USING btree (user_id, is_deleted)`
- `CREATE UNIQUE INDEX files_bucket_path_unique_idx ON public.files USING btree (storage_bucket, storage_path) WHERE (is_deleted = false)`

## Row estimate & freshness

Row estimate: 306

Freshness: facts reflect the snapshot recorded in `generated_at` (front-matter).

## Referenced-by

| Object | Via |
|---|---|
| [`supabase.public.cover_letter_exports`](cover_letter_exports.schema.md) | `file_id` |
| [`supabase.public.exports`](exports.schema.md) | `file_id` |
| [`supabase.public.imports`](imports.schema.md) | `source_file_id` |
