---
doc_class: machine-object
object: supabase.reporting.v_mart_dim_breakouts
kind: view
schema_hash: "sha256:3c9c774ef9a3f94a4aa89215020fe4ee1e2ce3a5d0e41dab50ef1f2ce83104ed"
generated_at: 2026-08-04
source_mode: live
snapshot_version: "1"
status: machine
---

# `supabase.reporting.v_mart_dim_breakouts`

## Identity

| Fact | Value |
|---|---|
| Object | `supabase.reporting.v_mart_dim_breakouts` |
| Kind | view |
| Schema hash | `sha256:3c9c774ef9a3f94a4aa89215020fe4ee1e2ce3a5d0e41dab50ef1f2ce83104ed` |

## Columns

| # | Column | Type | Nullable | Default | Description | Purpose |
|---|---|---|---|---|---|---|
| 1 | `dim` | `text` | true | — | — | — |
| 2 | `key1` | `text` | true | — | — | — |
| 3 | `key2` | `text` | true | — | — | — |
| 4 | `item_count` | `bigint` | true | — | — | — |
| 5 | `user_count` | `bigint` | true | — | — | — |
| 6 | `total_bytes` | `numeric` | true | — | — | — |
| 7 | `cancelling_count` | `bigint` | true | — | — | — |

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
 SELECT 'exports_by_format'::text AS dim,
    v_exports_by_format.format AS key1,
    v_exports_by_format.status AS key2,
    v_exports_by_format.export_count AS item_count,
    v_exports_by_format.distinct_users AS user_count,
    NULL::numeric AS total_bytes,
    NULL::bigint AS cancelling_count
   FROM reporting.v_exports_by_format
UNION ALL
 SELECT 'imports_by_parser'::text AS dim,
    v_imports_by_parser.parser_name AS key1,
    v_imports_by_parser.status AS key2,
    v_imports_by_parser.import_count AS item_count,
    v_imports_by_parser.distinct_users AS user_count,
    NULL::numeric AS total_bytes,
    NULL::bigint AS cancelling_count
   FROM reporting.v_imports_by_parser
UNION ALL
 SELECT 'files_by_type'::text AS dim,
    v_files_by_type.file_type AS key1,
    v_files_by_type.mime_type AS key2,
    v_files_by_type.file_count AS item_count,
    v_files_by_type.distinct_users AS user_count,
    v_files_by_type.total_bytes,
    NULL::bigint AS cancelling_count
   FROM reporting.v_files_by_type
UNION ALL
 SELECT 'master_cvs_by_language'::text AS dim,
    v_master_cvs_by_language.language AS key1,
    v_master_cvs_by_language.source_type AS key2,
    v_master_cvs_by_language.master_cv_count AS item_count,
    v_master_cvs_by_language.distinct_users AS user_count,
    NULL::numeric AS total_bytes,
    NULL::bigint AS cancelling_count
   FROM reporting.v_master_cvs_by_language
UNION ALL
 SELECT 'jobs_by_status'::text AS dim,
    v_jobs_by_status.status AS key1,
    NULL::text AS key2,
    v_jobs_by_status.job_count AS item_count,
    v_jobs_by_status.distinct_users AS user_count,
    NULL::numeric AS total_bytes,
    NULL::bigint AS cancelling_count
   FROM reporting.v_jobs_by_status
UNION ALL
 SELECT 'subscriptions_by_plan'::text AS dim,
    v_subscriptions_by_plan.plan_code AS key1,
    v_subscriptions_by_plan.status AS key2,
    v_subscriptions_by_plan.subscription_count AS item_count,
    NULL::bigint AS user_count,
    NULL::numeric AS total_bytes,
    v_subscriptions_by_plan.cancelling_count
   FROM reporting.v_subscriptions_by_plan;
```
