---
doc_class: machine-object
object: supabase.reporting.v_user_signups_by_day
kind: view
schema_hash: "sha256:cba88302f11c160a969970b5b4717baa13862a2ca7098bfcdfc504066d0a1da6"
generated_at: 2026-07-27
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
| Schema hash | `sha256:cba88302f11c160a969970b5b4717baa13862a2ca7098bfcdfc504066d0a1da6` |

## Columns

| # | Column | Type | Nullable | Default | Description | Purpose |
|---|---|---|---|---|---|---|
| 1 | `signup_day` | `date` | true | — | — | — |
| 2 | `new_users` | `bigint` | true | — | — | — |

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
 SELECT (created_at AT TIME ZONE 'UTC'::text)::date AS signup_day,
    count(*) AS new_users
   FROM public.users
  GROUP BY ((created_at AT TIME ZONE 'UTC'::text)::date);
```
