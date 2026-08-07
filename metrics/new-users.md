---
doc_class: metric
status: verified
last_verified: "2026-08-08 (Alper Camli)"
owner: "alper (operator)"
aliases: [signups, new signups, new accounts]
sources:
  - "customer-verified SQL: benchmark suite case RB-01 (.contextlayer/benchmark/suite.yaml)"
  - "machine doc: supabase.reporting.v_user_signups_by_day (view definition)"
  - "human doc: supabase.public.users"
depends_on:
  - supabase.public.users
  - supabase.reporting.v_user_signups_by_day
contamination: null
---

# new users

## Definition

**One `public.users` row, counted by `created_at`.** A person who created an
account in the window. `users` has no soft-delete column, so a signup once
counted stays counted; `created_at` is immutable, which is what makes this
metric byte-stable for a closed window.

## Formula

`count(*)` over `public.users` where `created_at` falls in the window, bucketed
by the UTC calendar day of `created_at`.

## Implementations

**`supabase` — the system of record.** Verbatim from the customer-verified
golden (RB-01):

```sql
SELECT date_trunc('day', created_at AT TIME ZONE 'UTC')::date AS signup_day,
       count(*) AS new_users
FROM public.users
WHERE created_at >= '<window_start>'
  AND created_at <  '<window_end>'
GROUP BY 1
ORDER BY 1;
```

**`supabase` — pre-aggregated, for reports that may not read base tables.**
`reporting.v_user_signups_by_day` is the same computation with no window:

```sql
SELECT signup_day, new_users FROM reporting.v_user_signups_by_day
WHERE signup_day >= '<window_start>' AND signup_day < '<window_end>';
```

The view is the route under RLS — the base table returns zero rows to the
execution role (see [`v_user_signups_by_day.md`](../systems/supabase/reporting/v_user_signups_by_day.md)).

## Grain & dimensions

Grain: one row per UTC signup day. Sliceable by nothing else in these two
implementations — locale and default CV language need
`reporting.v_user_cohorts` (monthly grain), which is a different bucket and
must not be summed against this one.

## Known discrepancies

- **GA4's `newUsers` is not this metric.** It counts first-seen *devices/
  identities* in analytics, includes people who never signed up, and misses
  signups by anyone blocking analytics. The two are compared deliberately in
  the funnel case (RB-05) as *stages*, never as the same number. See
  [`entities/user.md`](../entities/user.md) for the identity mapping and why
  no row-level join exists between them.
- The view has no window predicate, so a query without one scans the whole
  history; that is a cost matter, not a correctness one.

