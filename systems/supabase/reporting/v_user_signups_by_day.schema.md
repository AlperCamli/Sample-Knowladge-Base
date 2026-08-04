---
doc_class: machine-object
object: supabase.reporting.v_user_signups_by_day
kind: view
schema_hash: "sha256:59eaa3b44cc53232cb9e83609a01990fc26dfab632282b59e33473eedd498192"
generated_at: 2026-08-04
source_mode: live
snapshot_version: "1"
status: machine
---

# `supabase.reporting.v_user_signups_by_day`

## Identity

| Fact | Value |
|---|---|
| Object | `supabase.reporting.v_user_signups_by_day` |
| Kind | view |
| Schema hash | `sha256:59eaa3b44cc53232cb9e83609a01990fc26dfab632282b59e33473eedd498192` |

## Columns

| # | Column | Type | Nullable | Default | Description | Purpose |
|---|---|---|---|---|---|---|
| 1 | `signup_date` | `date` | true | — | — | UTC calendar date the accounts were created (`users.created_at` converted to UTC, then truncated). |
| 2 | `new_users` | `bigint` | true | — | — | Accounts created on that date. |

## Keys & indexes

Primary key: —

Foreign keys: —

Unique constraints: —

Indexes: —

## Row estimate & freshness

Row estimate: —

Freshness: facts reflect the snapshot recorded in `generated_at` (front-matter).

## Referenced-by

—

## View definition

```sql
 SELECT (created_at AT TIME ZONE 'UTC'::text)::date AS signup_date,
    count(*) AS new_users
   FROM public.users
  GROUP BY ((created_at AT TIME ZONE 'UTC'::text)::date);
```
