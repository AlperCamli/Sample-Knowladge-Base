---
doc_class: machine-object
object: supabase.reporting.v_activation_funnel_monthly
kind: view
schema_hash: "sha256:d1c38c2fe7c88702255c00612304dc05b7cc290661e4cf3a366e043a3f16bcf5"
generated_at: 2026-07-27
source_mode: live
snapshot_version: "1"
status: machine
---

# `supabase.reporting.v_activation_funnel_monthly`

## Identity

| Fact | Value |
|---|---|
| Object | `supabase.reporting.v_activation_funnel_monthly` |
| Kind | view |
| Schema hash | `sha256:d1c38c2fe7c88702255c00612304dc05b7cc290661e4cf3a366e043a3f16bcf5` |

## Columns

| # | Column | Type | Nullable | Default | Description | Purpose |
|---|---|---|---|---|---|---|
| 1 | `cohort_month` | `date` | true | — | — | — |
| 2 | `signed_up` | `bigint` | true | — | — | — |
| 3 | `created_master_cv` | `bigint` | true | — | — | — |
| 4 | `created_tailored_cv` | `bigint` | true | — | — | — |
| 5 | `exported` | `bigint` | true | — | — | — |
| 6 | `subscribed` | `bigint` | true | — | — | — |

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
 SELECT cohort_month,
    count(*) AS signed_up,
    count(*) FILTER (WHERE (EXISTS ( SELECT 1
           FROM public.master_cvs m
          WHERE m.user_id = u.id AND m.is_deleted = false AND (m.created_at AT TIME ZONE 'UTC'::text) < u.next_month))) AS created_master_cv,
    count(*) FILTER (WHERE (EXISTS ( SELECT 1
           FROM public.tailored_cvs t
          WHERE t.user_id = u.id AND t.is_deleted = false AND (t.created_at AT TIME ZONE 'UTC'::text) < u.next_month))) AS created_tailored_cv,
    count(*) FILTER (WHERE (EXISTS ( SELECT 1
           FROM public.exports e
          WHERE e.user_id = u.id AND e.status = 'completed'::text AND (e.created_at AT TIME ZONE 'UTC'::text) < u.next_month))) AS exported,
    count(*) FILTER (WHERE (EXISTS ( SELECT 1
           FROM public.subscriptions s
          WHERE s.user_id = u.id AND (s.created_at AT TIME ZONE 'UTC'::text) < u.next_month))) AS subscribed
   FROM ( SELECT users.id,
            date_trunc('month'::text, (users.created_at AT TIME ZONE 'UTC'::text))::date AS cohort_month,
            date_trunc('month'::text, (users.created_at AT TIME ZONE 'UTC'::text)) + '1 mon'::interval AS next_month
           FROM public.users) u
  GROUP BY cohort_month;
```
