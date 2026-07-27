---
doc_class: machine-object
object: supabase.reporting.v_subscriptions_new_by_month
kind: view
schema_hash: "sha256:a3fb70e6c96b954a9ea11e1935eb412a3807b9778d8b4d68e6dc6866f6bdb272"
generated_at: 2026-07-27
source_mode: live
snapshot_version: "1"
status: machine
---

# `supabase.reporting.v_subscriptions_new_by_month`

## Identity

| Fact | Value |
|---|---|
| Object | `supabase.reporting.v_subscriptions_new_by_month` |
| Kind | view |
| Schema hash | `sha256:a3fb70e6c96b954a9ea11e1935eb412a3807b9778d8b4d68e6dc6866f6bdb272` |

## Columns

| # | Column | Type | Nullable | Default | Description | Purpose |
|---|---|---|---|---|---|---|
| 1 | `month` | `date` | true | — | — | First day of the UTC month the subscription record was created in. |
| 2 | `status` | `text` | true | — | — | Lifecycle status of the record as it stands now, not as it was at creation. Open vocabulary — see Warnings. |
| 3 | `new_subscriptions` | `bigint` | true | — | — | Subscription records created that month with that current status. |

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
 SELECT date_trunc('month'::text, (created_at AT TIME ZONE 'UTC'::text))::date AS month,
    status,
    count(*) AS new_subscriptions
   FROM public.subscriptions
  GROUP BY (date_trunc('month'::text, (created_at AT TIME ZONE 'UTC'::text))::date), status;
```
